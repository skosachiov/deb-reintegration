## 1. Relocate git tasks to post_tasks

- [x] 1.1 In `playbooks/nvidia-graphics-drivers.yml`, remove the `pre_tasks` block containing the git include and add a `post_tasks` block that includes `tasks/git.yml` with `vars: { dsc_url: "{{ nvidia_dsc_url }}", build_root: "{{ corpos_build_root }}" }`. Verify: `ansible-playbook --syntax-check -i inventory.ini -e "@vars-nvidia.yml" playbooks/nvidia-graphics-drivers.yml` passes and the pre_tasks block is gone.

## 2. Genericize playbooks/tasks/git.yml

- [x] 2.1 Replace the variable preamble so the file reads only generic `dsc_url` and `build_root` inputs and `set_fact`s `dsc_file`, `source_dirname`, `build_dir`, `source_dir`, `package_version` (version extracted from `dsc_url` regex). Verify: `grep -c 'nvidia_\|corpos_' playbooks/tasks/git.yml` returns 0.
- [x] 2.2 Replace the empty-commit upstream branch with a real source commit: `git checkout --orphan upstream`, `git add .`, `git rm -r --cached debian/`, commit "Import upstream source", then tag `upstream/<version>`; relax `failed_when` so a re-run over an existing upstream branch/tag is tolerated (mirror the "already exists" handling already used for tags). Verify: file contains an `upstream` orphan creation with `git rm -r --cached debian/`, a commit, and the `upstream/{{ package_version }}` tag.
- [x] 2.3 Keep the `debian/gbp.conf` branch read (default `debian/master`) and the checkout/add/commit of `debian/` plus the `debian/<version>` tag, with re-run-tolerant `failed_when`. Verify: file references `debian_branch`, `git add debian/`, and `debian/{{ package_version }}`.

## 3. Make the nvidia role self-sufficient on path variables

- [x] 3.1 Extend the "Set DSC-related variables" task in `roles/nvidia-graphics-drivers/tasks/download.yml` to also `set_fact` `dsc_file` (`nvidia_dsc_url | basename`), `source_dirname` (`(nvidia_dsc_url | basename) | splitext | first`), and `source_dir` (`nvidia_build_dir + "/" + source_dirname`). Verify: `unpack.yml`/`modify.yml` no longer depend on facts that only git.yml set — checked via a dry run (`--check`) reaching the unpack step without an undefined-variable error.

## 4. End-to-end validation

- [x] 4.1 Run the full playbook (real run, not check) against the local builder; verify the resulting git-buildpackage repo under the build root (module CWD is `playbooks/`, so `playbooks/build/...`) has: an `upstream` branch whose tree contains the source but not `debian/`, tagged `upstream/<version>`; a debian branch (name from `debian/gbp.conf`) containing the source plus committed `debian/`, tagged `debian/<version>`; and that `git ls-tree` output confirms both trees.