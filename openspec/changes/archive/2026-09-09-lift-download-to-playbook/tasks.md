## 1. Create shared download task file

- [x] 1.1 Create `playbooks/tasks/download.yml` with the download logic extracted from `roles/nvidia-graphics-drivers/tasks/download.yml`, replacing all `nvidia_*` variable names with generic equivalents (`dsc_url`, `build_dir`, `build_root`, `base_url`, `source_files`). Verify: the file exists and contains all task blocks from the original download.yml with generic variable names only.

## 2. Update playbook to include shared download tasks

- [x] 2.1 Edit `playbooks/nvidia-graphics-drivers.yml` to add `include_tasks: tasks/download.yml` in `pre_tasks` with variable mapping (`dsc_url: "{{ nvidia_dsc_url }}"`, `build_root: "{{ corpos_build_root }}"`, `source_files: "{{ nvidia_source_files | default([]) }}"`). Verify: `ansible-playbook --syntax-check -i inventory.ini -e "@vars-nvidia.yml" playbooks/nvidia-graphics-drivers.yml` passes.

## 3. Remove download logic from the nvidia role

- [x] 3.1 Remove the `Download sources` include step (`include_tasks: download.yml`) from `roles/nvidia-graphics-drivers/tasks/main.yml`. Verify: `main.yml` no longer references `download.yml`.
- [x] 3.2 Delete `roles/nvidia-graphics-drivers/tasks/download.yml`. Verify: the file no longer exists on disk.
