# Spec 002 - nvidia-graphics-drivers: Ubuntu driver rebuild for Debian

## Status
Proposed

## Problem

The `nvidia-graphics-drivers-595` package is officially packaged only for
Ubuntu (in the `restricted` component). We want to build it as a native Debian
package. The Ubuntu source carries Ubuntu-specific packaging (`dh-modaliases`
build dep, `dh_modaliases` bytecode) and a version string tied to the Ubuntu
archive naming, which must be normalized for a clean Debian build.

## Upstream Sources

Fetched from the Ubuntu `restricted` pool:

| Component | URL |
| --------- | --- |
| Upstream (unpacked) tree, amd64 | `http://archive.ubuntu.com/ubuntu/pool/restricted/n/nvidia-graphics-drivers-595/nvidia-graphics-drivers-595_595.84.orig-amd64.tar.gz` |
| Upstream (unpacked) tree, arm64 | `http://archive.ubuntu.com/ubuntu/pool/restricted/n/nvidia-graphics-drivers-595/nvidia-graphics-drivers-595_595.84.orig-arm64.tar.gz` |
| Upstream tar (packaged) | `http://archive.ubuntu.com/ubuntu/pool/restricted/n/nvidia-graphics-drivers-595/nvidia-graphics-drivers-595_595.84.orig.tar.gz` |
| Debian metadata | `http://archive.ubuntu.com/ubuntu/pool/restricted/n/nvidia-graphics-drivers-595/nvidia-graphics-drivers-595_595.84-0ubuntu0.26.04.1.debian.tar.xz` |
| Source control file | `http://archive.ubuntu.com/ubuntu/pool/restricted/n/nvidia-graphics-drivers-595/nvidia-graphics-drivers-595_595.84-0ubuntu0.26.04.1.dsc` |

Pinned: version `595.84`, Ubuntu revision `0ubuntu0.26.04.1`.

## Build Steps

The adapted source tree is produced in `build/nvidia-graphics-drivers-595/` by
the following steps:

### 1. Download sources

Download all five files listed above into
`build/nvidia-graphics-drivers-595/` (download area), preferably via
`get_url` with the `dsc`, `debian.tar.xz` and the three `orig` tarballs.

### 2. Unpack with `dpkg-source -x`

Run `dpkg-source -x nvidia-graphics-drivers-595_595.84-0ubuntu0.26.04.1.dsc`
in the download area. This yields the unpacked source tree
`nvidia-graphics-drivers-595-595.84/` containing the `debian/` control metadata
and rules.

### 3. Strip Ubuntu-specific packaging

- In `debian/control`: remove the Ubuntu-only `dh-modaliases` line from
  `Build-Depends`.
- In `debian/rules`: remove the Ubuntu-only `dh_modaliases` invocation
  (the `--with modaliases` dh addon) from the build rules.

These make the package buildable without the Ubuntu `modaliases` machinery.

### 4. Version the Debian build

Append the `+0corpos1` suffix to the current package version in
`debian/changelog` so the resulting build is clearly identified as a corpos
Debian rebuild of the Ubuntu source (e.g. `595.84-0ubuntu0.26.04.1+0corpos1`).
The `+0` keeps the resulting version below any future Ubuntu update while
`corpos1` marks our revision.

### 5. Test build ability

Run `dpkg-buildpackage -us -uc -b` inside the adapted source tree to verify the
package builds cleanly. This is a build-ability smoke test; whether artifacts
are kept or the folder is discarded is captured in the acceptance criteria.

## Result

`build/nvidia-graphics-drivers-595/` contains:

- the downloaded source files,
- the unpacked, adapted source tree `nvidia-graphics-drivers-595-595.84/` that a
  `dpkg-buildpackage` run consumes,
- the build artifacts produced by the build-ability test.

## Acceptance Criteria

- [ ] All five source files are downloaded under `build/nvidia-graphics-drivers-595/`.
- [ ] `dpkg-source -x` succeeds and yields the unpacked source tree.
- [ ] No `dh-modaliases` / `dh_modaliases` references remain in the adapted tree.
- [ ] `debian/changelog` version carries the `+0corpos1` suffix.
- [ ] `dpkg-buildpackage -us -uc -b` completes successfully inside the adapted tree.