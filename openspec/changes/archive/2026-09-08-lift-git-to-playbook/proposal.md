## Why

The git-buildpackage initialization logic (git init, upstream/debian branches, tags, initial commit) lives inside `roles/nvidia-graphics-drivers/tasks/git.yml`. This couples a reusable packaging workflow to a single role. Any future role that repackages Ubuntu sources for Debian would need to duplicate this logic. Lifting it to the playbook level makes it a shared concern available to all roles.

## What Changes

- Extract `git.yml` task logic from `roles/nvidia-graphics-drivers/` to a top-level reusable location (e.g., `playbooks/tasks/git.yml` or a shared tasks directory)
- Update `playbooks/nvidia-graphics-drivers.yml` to include the extracted tasks before invoking the role
- Remove the git-related tasks from `roles/nvidia-graphics-drivers/tasks/main.yml` and `roles/nvidia-graphics-drivers/tasks/git.yml`
- Variables consumed by git tasks (`nvidia_build_dir`, `nvidia_dsc_url`, etc.) must still be available at include time; the role currently sets some of these in `main.yml` before including `git.yml`, so the extraction must preserve that variable setup or move it upstream as well

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

(none)

## Impact

- **Files modified**: `roles/nvidia-graphics-drivers/tasks/main.yml`, `playbooks/nvidia-graphics-drivers.yml`
- **Files removed**: `roles/nvidia-graphics-drivers/tasks/git.yml`
- **Files created**: new shared task file for git initialization
- **No external dependencies** change; the same git commands and Ansible modules are used
- **Behavior is unchanged** from the user's perspective — the playbook still produces the same git-buildpackage structure in the build directory
