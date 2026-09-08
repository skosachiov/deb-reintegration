## Why

The shared git-buildpackage tasks in `playbooks/tasks/git.yml` (lifted out of the nvidia role in a prior change) still reference nvidia-specific variables (`nvidia_dsc_url`, `nvidia_build_dir`, `nvidia_version`), so the file is not actually reusable for other packages. Worse, they run in `pre_tasks` — before download/unpack/strip/modify — so the `debian/` folder doesn't exist yet when `git add debian/` runs, and the `upstream/`/`debian/` tags point at an empty, unfilled tree. The user wants git to run *after* the packaging work ("after filling") so the branches and tags carry real content, and the task file to be package-agnostic.

## What Changes

- **Move git tasks from `pre_tasks` to `post_tasks`** in `playbooks/nvidia-graphics-drivers.yml`, so git-buildpackage initialization runs after download/unpack/strip/modify have filled the source tree.
- **Make `playbooks/tasks/git.yml` package-agnostic**: replace `nvidia_dsc_url`/`nvidia_build_dir`/`nvidia_version` with generic inputs (`dsc_url`, `build_root`) and computed facts (`dsc_file`, `source_dirname`, `build_dir`, `source_dir`, `package_version`). No `nvidia_*` variables remain in the file.
- **Full git-buildpackage content**: the unpacked upstream source is now committed to the `upstream` branch and tagged `upstream/<version>` (no longer an empty placeholder); the `debian/` packaging is committed on the debian branch (read from `debian/gbp.conf`, default `debian/master`) and tagged `debian/<version>`. This fixes the pre-existing fresh-run failure of `git add debian/`.
- **Role becomes self-sufficient for path variables**: with the git preamble no longer running first, the nvidia role must compute `source_dir`, `dsc_file`, `source_dirname` itself before `unpack.yml`/`modify.yml` use them.
- **BREAKING (internal)**: the pre_tasks include and its side-effect facts are removed; roles/reviewers that relied on git outputs (`source_dir`, `git_env`, `debian_branch`, `nvidia_version`) in pre_tasks must read the new variable contract.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

(none)

## Impact

- **Files modified**: `playbooks/nvidia-graphics-drivers.yml`, `playbooks/tasks/git.yml`, `roles/nvidia-graphics-drivers/tasks/download.yml`
- **Files created**: none (task file edited in place)
- **Behavior change**: the produced git-buildpackage repo now contains committed upstream source on `upstream/<version>` and `debian/<version>` tags pointing at the filled packaging; previously the upstream tag was empty and the debian branch could not be committed on a fresh run.
- **No new external dependencies**; same git commands and Ansible modules.