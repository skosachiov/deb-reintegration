## Change - Add nvidia-graphics-drivers subproject

### Motivation

Kick off the first concrete subproject following Spec 001: adapt the Ubuntu
`nvidia-graphics-drivers-595` package for building on/for Debian, fully
contained under `build/`.

### Specs affected

- [Spec 002 - nvidia-graphics-drivers](../../spec/nvidia-graphics-drivers/spec.md)

### Implementation

- Pinned sources: version `595.84`, Ubuntu revision `0ubuntu0.26.04.1`.
- Work area: `build/nvidia-graphics-drivers-595/`.
- Steps:
  1. Download the `dsc`, `debian.tar.xz` and three `orig` tarballs from the
     Ubuntu restricted pool.
  2. Unpack with `dpkg-source -x`.
  3. Strip Ubuntu-specific `dh-modaliases` (control) and `dh_modaliases` (rules).
  4. Append `+0corpos1` to the changelog version.
  5. Verify with `dpkg-buildpackage -us -uc -b`.

### Status

Proposed — spec documentation only; Ansible role to follow in a later change.
