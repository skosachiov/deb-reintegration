# Spec 001 - Corpos: Adapting non-Debian packages for Debian with Ansible

## Status
Proposed

## Problem

The project `corpos` adapts software packages not originally packaged for Debian
so they can be built and installed as native Debian packages. This is achieved
by driving the packaging workflow with Ansible automation.

Building Debian packages uses `dpkg-buildpackage`, which expects a particular
source tree layout (`debian/` control metadata, orig tarballs, patches, etc.)
and produces a set of build artifacts. Without strict conventions, build
scratch space, intermediate state and final artifacts get scattered across the
workspace, making builds non-reproducible and hard to clean up.

## Background

Adapting a non-Debian package involves:

1. Fetching upstream sources and/or existing Debian packaging metadata.
2. Decomposing them with `dpkg-source -x` into an unpacked source tree.
3. Installing build dependencies (`mk-build-deps`, `apt-get build-dep`).
4. Applying packaging changes and rules.
5. Running `dpkg-buildpackage -us -uc -b` to produce `.deb` artifacts.

This is orchestrated by Ansible playbooks so the procedure is declarative,
repeatable and auditable.

## Goals

- Provide a well-defined, deterministic build environment.
- Keep all build data confined to a single `build/` directory.
- Make the result of each build a folder that `dpkg-buildpackage` can run in.
- Allow multiple packages / architectures to be built independently and reliably.
- Document packaging rules as high-level, reviewable specifications.

## Non-Goals

- Not a replacement for upstream maintainership or a full Debian archive.
- Not responsible for signing packages or producing a repository index.

## High-Level Rules

### 1. Everything under `build/`

All build data and artifacts MUST be stored under the `build/` directory
(already ignored by git via `.gitignore`). No build output may be written
outside `build/`.

### 2. One build folder per `dpkg-buildpackage` run

The result of adapting a package is a folder inside `build/` that is a valid
`dpkg-buildpackage` source tree (contains `debian/` metadata, orig sources and
Debian revisions). The playbook's final step produces this folder, which is
ready to be subjected to `dpkg-buildpackage -us -uc -b`.

### 3. Ansible is the single automation entry point

All steps — fetching, unpacking, dependency install, patching and finalizing
the build folder — MUST be expressed as Ansible playbooks/roles. Nothing that
affects the build tree may be done manually outside the playbook.

### 4. Each package has its own role and spec

Each adapted package has a dedicated Ansible role and a corresponding Spec in
`openspec/spec/` that describes its source, options and resulting build folder.

### 5. Reproducibility

Playbooks MUST be deterministic: pin sources/versions, avoid ambient state, and
make the resulting `build/` tree self-contained.

### 6. Clean separation

`build/` is disposable scratch; the source of truth for packaging logic lives
in the playbooks and specs. `rm -rf build/<pkg>` and re-running the playbook
MUST yield the same result.

## Open Questions

- Whether to store built `.deb` artifacts inside the per-package build folder
  or in a shared `build/artifacts/` directory.

## Acceptance Criteria

- [ ] Running the playbook for a package produces a `build/<pkg>/` folder that
      `dpkg-buildpackage` accepts.
- [ ] All intermediate and final data exists only under `build/`.
- [ ] The procedure is fully expressed as Ansible with no manual steps.
