## Why

The Debian source download logic (DSC file fetch, source file extraction, source archive download) lives inside `roles/nvidia-graphics-drivers/tasks/download.yml` with nvidia-specific variable names. This couples a reusable packaging workflow to a single role. Any future role that repackages Ubuntu sources for Debian would need to duplicate this logic. Lifting it to the playbook level and generalizing variable names makes it a shared concern available to all roles.

## What Changes

- Extract `download.yml` task logic from `roles/nvidia-graphics-drivers/tasks/` to `playbooks/tasks/download.yml`, following the same pattern used for `git.yml`
- Replace nvidia-specific variable names (`nvidia_dsc_url`, `nvidia_build_dir`, `nvidia_base_url`, `nvidia_source_files`, etc.) with generic equivalents (`dsc_url`, `build_dir`, `base_url`, `source_files`)
- Update `playbooks/nvidia-graphics-drivers.yml` to include the shared download tasks before invoking the role
- Remove `include_tasks: download.yml` from `roles/nvidia-graphics-drivers/tasks/main.yml`
- Update nvidia role defaults and vars to map generic variable names or set them from existing nvidia values

## Capabilities

### New Capabilities

- `download`: Shared Debian source download logic — fetches DSC file, extracts source file list, downloads source tarballs from any Debian/Ubuntu source package URL

### Modified Capabilities

(none)

## Impact

- **Files modified**: `playbooks/nvidia-graphics-drivers.yml`, `roles/nvidia-graphics-drivers/tasks/main.yml`, `roles/nvidia-graphics-drivers/defaults/main.yml`
- **Files created**: `playbooks/tasks/download.yml`
- **Files removed**: `roles/nvidia-graphics-drivers/tasks/download.yml`
- **No external dependencies** change; the same Ansible modules (`get_url`, `set_fact`, `shell`) are used
- **Behavior is unchanged** — the playbook still downloads the same source files; only variable names and file location change
