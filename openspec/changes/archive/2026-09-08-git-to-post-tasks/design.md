## Context

`playbooks/tasks/git.yml` is currently included from `pre_tasks` in `playbooks/nvidia-graphics-drivers.yml`. It still reads nvidia-prefixed variables (`nvidia_dsc_url`, `nvidia_build_dir`, `nvidia_version`) and is written for the empty/placeholder git-buildpackage setup (empty upstream commit, `git add debian/` on a not-yet-unpacked tree). The role's `unpack.yml` and `modify.yml` depend on `source_dir`, `dsc_file`, `source_dirname`, which today come only from the git preamble's `set_fact`. See proposal.md - Why for motivation.

## Goals / Non-Goals

**Goals:**
- Run git-buildpackage initialization in `post_tasks`, so branches/tags are created from the filled source tree
- Make `playbooks/tasks/git.yml` package-agnostic (no `nvidia_*` variables) so any package playbook can reuse it
- Commit real upstream source to `upstream/<version>` and real packaging to the debian branch tagged `debian/<version>`
- Keep the nvidia role self-sufficient: it computes its own path variables before `unpack`

**Non-Goals:**
- Renaming the nvidia role's variables or making the role package-agnostic — the role stays nvidia-specific
- Changing gbp branch/tag naming semantics (`upstream/<version>`, `debian/<version>`, branch name from `debian/gbp.conf`)
- Touching `build.yml` or the download/unpack/strip/modify logic beyond the variable prerequisite
- Pushing to remotes or anything beyond local repo creation

## Decisions

### 1. Variable contract for the shared git tasks

`playbooks/tasks/git.yml` accepts exactly two generic inputs and derives everything else:

| Input | Supplied by playbook |
|---|---|
| `dsc_url` | `{{ nvidia_dsc_url }}` (extra var) |
| `build_root` | `{{ corpos_build_root }}` (role default, play-wide) |

The preamble `set_fact`s `dsc_file`, `source_dirname`, `build_dir`, `source_dir`, `package_version`, `git_env`, `debian_branch`. No `nvidia_*` or `corpos_*` name appears in the file.

The playbook maps the names at the include site so the shared file stays generic:

```yaml
roles:
  - role: nvidia-graphics-drivers
post_tasks:
  - name: Git-buildpackage finalization
    ansible.builtin.include_tasks: tasks/git.yml
    vars:
      dsc_url: "{{ nvidia_dsc_url }}"
      build_root: "{{ corpos_build_root }}"
```

**Alternative considered**: git.yml reads `nvidia_dsc_url` directly (no mapping). Rejected — that is exactly the coupling the user wants removed.

### 2. Role computes its own path variables

Because the pre_tasks git preamble disappears, `unpack.yml` and `modify.yml` would hit undefined `source_dir`/`dsc_file`/`source_dirname`. `download.yml` already derives `nvidia_build_dir`, `nvidia_dsc_dir`, `nvidia_dsc_filename`, `nvidia_package_name` in its "Set DSC-related variables" task, which runs before unpack — so that task is extended to also set `dsc_file`, `source_dirname`, `source_dir` from `nvidia_dsc_url`/`nvidia_build_dir`. The role keeps its nvidia-prefixed variables elsewhere; this only restores the generic path facts it consumed previously.

### 3. Full git-buildpackage content in git.yml

Sequence after the tree is filled (upstream source + `debian/` present):

```
git init
git checkout --orphan upstream
git add .                      # stage whole tree
git rm -r --cached debian/     # drop packaging from upstream only
git commit -m "Import upstream source"
git tag -a upstream/<version>
git checkout -b <debian-branch>   # from upstream HEAD
git add debian/
git commit -m "Initial debian packaging"
git tag -a debian/<version>
```

`<debian-branch>` comes from `debian/gbp.conf` (`debian-branch=` key), defaulting to `debian/master` — that file is copied into the tree by `modify.yml` ("Copy gbp.conf ... to debian folder"). The debian branch therefore contains the full source plus the packaging; the upstream branch contains source without `debian/`.

**Alternative considered**: keep the current empty-commit placeholder and only relocate the tasks. Rejected — the user wants upstream to carry the real source (full gbp).

**Idempotency**: existing tag tasks already tolerate "already exists". The orphan/branch creation gets the same `failed_when` relaxation for re-runs. Net effect: a fresh unpack (version change → `unpack.yml` wipes `source_dir`) is followed by a clean repo rebuild; a no-op re-run over an already-built repo tolerates existing branches/tags.

## Risks / Trade-offs

- **Large upstream source committed** (NVIDIA build trees run to hundreds of MB) → Accepted tradeoff of the full-gbp choice; the repo lives under the gitignored `build_root`, so it is never committed to this Ansible repo nor pushed.
- **`git rm -r --cached debian/` fails if `debian/` absent** → `debian/` is guaranteed present because git runs in `post_tasks` after `unpack.yml`; add a guard/`failed_when` to surface the broken precondition loudly rather than misbehave.
- **Variable formulas duplicated between role and git.yml** → Both derive paths from the same DSC URL; the role's computation is authoritative during its phase, git.yml during post. Document the path formulas in git.yml's header comment so a change to one is mirrored in the other.
- **Binaries/artifacts accidentally committed to upstream branch if upstream has no `.gitignore`** → Existing behavior for the working tree; acceptable for a local build repo.

## Migration Plan

Edit three files: change the playbook `pre_tasks` block to a `post_tasks` block with the vars mapping; rewrite `playbooks/tasks/git.yml` (generic preamble + full-gbp sequence); extend `download.yml`'s "Set DSC-related variables" task. Rollback is a revert of these three edits (the prior change is archived).