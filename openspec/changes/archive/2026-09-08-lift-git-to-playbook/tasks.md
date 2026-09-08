## 1. Create shared git task file

- [x] 1.1 Create `playbooks/tasks/git.yml` with the git-buildpackage initialization logic extracted from `roles/nvidia-graphics-drivers/tasks/git.yml`, including the variable computation preamble (`dsc_file`, `source_dirname`, `source_dir`) that the git tasks consume. Verify: the file exists and contains all 10 task blocks from the original git.yml plus the 3 `set_fact` tasks from main.yml lines 1-5.

## 2. Update playbook to include shared git tasks

- [x] 2.1 Edit `playbooks/nvidia-graphics-drivers.yml` to add `include_tasks: tasks/git.yml` before the role invocation. Verify: the playbook YAML is valid (`ansible-playbook --syntax-check -i inventory.ini -e "@vars-nvidia.yml" playbooks/nvidia-graphics-drivers.yml`).

## 3. Remove git logic from the nvidia role

- [x] 3.1 Remove the `Git init` include step (`include_tasks: git.yml`) from `roles/nvidia-graphics-drivers/tasks/main.yml`. Verify: `main.yml` no longer references `git.yml`.
- [x] 3.2 Remove the `Calculate and set driver source variables` task block (lines 1-5 of main.yml) since those variables are now computed in the shared git task file. Verify: main.yml starts with `Ensure build directory exists`.
- [x] 3.3 Delete `roles/nvidia-graphics-drivers/tasks/git.yml`. Verify: the file no longer exists on disk.

## 4. End-to-end validation

- [x] 4.1 Run `ansible-playbook --syntax-check -i inventory.ini -e "@vars-nvidia.yml" playbooks/nvidia-graphics-drivers.yml` and verify no errors. Then run the full playbook with `--check` (dry run) to confirm task inclusion order is correct and no undefined variable errors occur.
