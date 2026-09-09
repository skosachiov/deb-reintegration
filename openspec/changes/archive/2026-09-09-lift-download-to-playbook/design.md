## Context

The nvidia role's `main.yml` orchestrates download, unpack, strip, modify, and build steps. The download step (`download.yml`) fetches a DSC file, extracts source file references from it, and downloads all source tarballs. This logic is reusable for any Debian/Ubuntu source package repackaging workflow, but is currently coupled to nvidia-specific variable names (`nvidia_dsc_url`, `nvidia_build_dir`, `nvidia_base_url`, `nvidia_source_files`).

The same refactoring pattern was already applied to `git.yml` in a prior change — the git-buildpackage init logic was lifted from the role to `playbooks/tasks/git.yml`. This change follows the identical approach for download.

Key variable dependencies in current `download.yml`:
- `nvidia_dsc_url` — DSC file URL (from defaults)
- `nvidia_build_dir` — target directory (computed in main.yml)
- `nvidia_base_url` — base URL for source files (derived from `nvidia_dsc_url | dirname`)
- `nvidia_source_files` — optional additional source files list (from defaults, defaults to empty)
- `corpos_build_root` — parent build root (from defaults)

## Goals / Non-Goals

**Goals:**
- Make source download a shared, reusable task file accessible from any playbook
- Replace all `nvidia_*` variable names with generic equivalents (`dsc_url`, `build_dir`, `base_url`, `source_files`, etc.)
- Preserve identical download behavior — same DSC fetch, same source extraction, same file downloads
- Keep the nvidia role focused on nvidia-specific packaging concerns (unpack, strip, modify)

**Non-Goals:**
- Refactoring unpack, strip, modify, or build tasks — they remain role-specific
- Changing the download mechanism itself (same `get_url`, same `awk` parsing, same flow)
- Adding caching, retry logic, or parallel downloads
- Modifying the git.yml shared task or its variable conventions

## Decisions

### 1. Shared task file location: `playbooks/tasks/download.yml`

**Chosen**: `playbooks/tasks/download.yml`

Same location as the shared `git.yml`. The playbook includes it from `post_tasks` or `pre_tasks` depending on ordering requirements.

**Alternatives considered**:
- Same approach already validated for `git.yml` — no new alternatives needed

### 2. Variable naming convention

**Chosen**: Generic names matching the pattern established by `git.yml`:
- `dsc_url` (was `nvidia_dsc_url`)
- `build_dir` (was `nvidia_build_dir`)
- `build_root` (was `corpos_build_root`)
- `base_url` (was `nvidia_base_url`, derived from dsc_url dirname)
- `source_files` (was `nvidia_source_files`)

The shared task computes all derived variables (`package_name`, `dsc_filename`, `dsc_dir`, `source_dirname`, `source_dir`) internally via `set_fact`, same as the current implementation.

**Alternatives considered**:
- Keeping nvidia-prefixed names — rejected because it defeats the purpose of generalization
- Using a namespace/prefix parameter — over-engineered for this project's scale

### 3. Include mechanism: `post_tasks`

**Chosen**: Include the shared download tasks in the playbook's `post_tasks` section, after the role completes its variable setup.

The download needs variables that the nvidia role's defaults provide (`dsc_url`, `build_root`, `source_files`). Since role defaults are loaded for the whole play when the role is declared, these variables are in scope during `post_tasks`. The download tasks then compute derived variables (`build_dir`, `package_name`, `source_dir`) via `set_fact`, making them available to subsequent role tasks if needed.

However, looking at the current flow more carefully: the download step must happen **before** unpack, strip, modify. The current `main.yml` includes download as the first step after ensuring the build directory. If download moves to `post_tasks`, it would run **after** the role — breaking the flow.

**Revised approach**: Keep the download include **inside** the role's `main.yml` but source it from the shared location, OR include it in `pre_tasks` before the role. Since the role's `main.yml` currently orchestrates the sequence (download → unpack → strip → modify → build), the cleanest approach is:

- The playbook includes shared download tasks in `pre_tasks` (before the role)
- The role's `main.yml` removes the `Download sources` include step
- The role's remaining tasks (unpack, strip, modify, build) rely on variables set by the shared download tasks

This matches the `git.yml` pattern exactly.

### 4. Variable mapping in nvidia defaults

**Chosen**: The nvidia role's `defaults/main.yml` keeps its existing variable names and the playbook maps them when invoking the shared tasks:

```yaml
pre_tasks:
  - name: Download sources
    ansible.builtin.include_tasks: tasks/download.yml
    vars:
      dsc_url: "{{ nvidia_dsc_url }}"
      build_root: "{{ corpos_build_root }}"
      source_files: "{{ nvidia_source_files | default([]) }}"
```

This preserves backward compatibility — existing vars files (`vars-nvidia.yml`) continue to work unchanged.

## Risks / Trade-offs

- **Variable scope leakage** — The shared download tasks set `build_dir`, `source_dir`, `dsc_file`, etc. as `set_fact`. These persist in the play context and remain available to downstream role tasks. This is the desired behavior but couples the caller to the shared task's variable names. → Mitigate by documenting the expected inputs and produced outputs in the shared task file's header comment.

- **Role becomes less self-contained** — Running the nvidia role alone would no longer include download. → Acceptable: same trade-off accepted for git.yml; the role is always invoked from the playbook.

- **Ordering sensitivity** — Download must happen before unpack. The playbook's `pre_tasks` → `roles` ordering enforces this. → Low risk; same pattern validated with git.yml.
