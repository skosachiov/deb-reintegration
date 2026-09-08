# Specifications Catalog

## Spec 002 - nvidia-graphics-drivers: Ubuntu driver rebuild for Debian

[spec](../spec/nvidia-graphics-drivers/spec.md)

| Field | Value |
| ----- | ----- |
| Status | Proposed |
| Area | Packaging: nvidia-graphics-drivers-595 |
| Spec  | 002 |

First concrete subproject of `corpos`. Adapts the Ubuntu
`nvidia-graphics-drivers-595` (version `595.84`, revision
`0ubuntu0.26.04.1`) package for a Debian build:

- Download the five pool source files.
- Unpack with `dpkg-source -x`.
- Strip Ubuntu-only `dh-modaliases` / `dh_modaliases`.
- Version the changelog with the `+0corpos1` suffix.
- Smoke-test with `dpkg-buildpackage -us -uc -b`.

All work happens under `build/nvidia-graphics-drivers-595/`.

See also [Spec 001 - High-level rules](../overview/catalog.md).
