## Change - Add OpenSpec base structure and project conventions

### Motivation

Establish a reviewable specification layer for how `corpos` adapts non-Debian
packages for Debian with Ansible, and codify the core build rules.

### Specs affected

- [Base high-level rules](../../spec/overview/spec.md)

### Implementation

- Created `openspec/` folder structure (`spec/`, `catalog/`, `changes/`, `projects/`).
- Added Spec 001 defining the high-level build and packaging rules:
  - All build data is stored under `build/`.
  - The result is a folder `dpkg-buildpackage` can run in.
  - Ansible is the single automation entry point.
  - Each package has its own role and spec, build is reproducible, `build/` is disposable.

### Status

Proposed — awaiting review and approval.
