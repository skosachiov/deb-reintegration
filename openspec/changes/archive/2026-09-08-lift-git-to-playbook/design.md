## Context

The nvidia role's `main.yml` orchestrates six task files in sequence: variable setup, git init, download, unpack, strip, modify, build. The git step (`git.yml`) sets up git-buildpackage structure — this is a reusable packaging concern, not nvidia-specific. It currently lives at `roles/nvidia-graphics-drivers/tasks/git.yml` and is included via `include_tasks: git.yml`.

Key variable dependencies the git tasks consume:
- `nvidia_build_dir` — set in `main.yml` line 5, used as `chdir` target
- `nvidia_dsc_url` — set in defaults, used to extract version and compute paths
- `source_dir` — set in `main.yml` line 5, target for git init and commits
- `corpos_build_root` — set in defaults, parent of build dir

## Goals / Non-Goals

**Goals:**
- Make git-buildpackage initialization a shared, reusable task file accessible from any playbook
- Preserve identical behavior — same git commands, same branch/tag structure, same commit content
- Keep the nvidia role focused on nvidia-specific packaging concerns (download, unpack, strip, modify)

**Non-Goals:**
- Refactoring the other task files (download, unpack, etc.) — they are role-specific
- Changing the git workflow itself (branch names, tag format, commit messages)
- Adding new capabilities or variables beyond what's needed for extraction
- Modifying the build.yml stub

## Decisions

### 1. Shared task file location: `playbooks/tasks/git.yml`

**Chosen**: `playbooks/tasks/git.yml`

**Alternatives considered**:
- `roles/common/tasks/git.yml` — requires creating a new role for one file; over-engineered
- `tasks/git.yml` at repo root — non-standard Ansible layout
- A collection — far too heavy for this project

`playbooks/tasks/` follows Ansible convention for task files shared across playbooks. The playbook includes them with a relative path: `include_tasks: ../playbooks/tasks/git.yml` (from role context) or directly from the playbook.

### 2. Variable ownership

The git tasks need `source_dir`, `nvidia_build_dir`, and `nvidia_dsc_url`. Currently `main.yml` computes `source_dir` and `dsc_file` before including git.yml. 

**Chosen**: Move the variable computation (`source_dir`, `dsc_file`, `source_dirname`) into the shared git task file as a preamble. This makes the shared tasks self-contained — any caller only needs to provide `nvidia_dsc_url` and `corpos_build_root`.

The nvidia role's `main.yml` will then only set `nvidia_build_dir` (which it already does in the `download.yml` include) and remove the redundant variable setup that git no longer needs.

### 3. Include mechanism: `pre_tasks`

**Chosen**: The playbook `playbooks/nvidia-graphics-drivers.yml` includes the shared git tasks in a `pre_tasks:` block, which Ansible executes **before** the `roles:` section:

```yaml
pre_tasks:
  - name: Git-buildpackage initialization
    ansible.builtin.include_tasks: tasks/git.yml
roles:
  - role: nvidia-graphics-drivers
```

**Why not `tasks:`**: Ansible applies `roles:` before the play's `tasks:` section regardless of YAML declaration order. A plain `tasks:` include would run git init *after* download/unpack/strip/modify — breaking the required ordering (git init must precede the role). The `--check` dry run confirmed this bug before the fix.

**Critical implication**: role default variables (`corpos_build_root`, `nvidia_dsc_url`) are loaded for the whole play as soon as the role is declared, so they are in scope during `pre_tasks`. The shared git file's preamble computes `nvidia_build_dir`, `dsc_file`, `source_dirname`, and `source_dir` on its own via `set_fact`, so it only depends on `nvidia_dsc_url` and `corpos_build_root`.

The nvidia role's `main.yml` removes the `Git init` include step. Download, unpack, strip, modify, build remain in the role.

## Risks / Trade-offs

- **Variable scope leakage** — The shared git tasks compute variables (`dsc_file`, `source_dirname`, `source_dir`) that downstream role tasks also use. If the shared tasks set these as `set_fact`, they persist in the play context and remain available to the role. This is the desired behavior but couples the caller to the shared task's variable names. → Mitigate by documenting the expected variables in the shared task file's header comment.

- **Role becomes less self-contained** — Running the nvidia role alone (without the playbook) would no longer include git setup. → Acceptable: the role was never designed for standalone use; it's always invoked from the playbook.

- **Ordering sensitivity** — Git init must happen before download and unpack. The playbook's include order enforces this. → Low risk; the existing flow already depends on this ordering.
