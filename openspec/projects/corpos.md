# Project: Corpos base conventions

This project adapts non-Debian packages for Debian using Ansible.

## Goals

A repeatable, fully-Ansible-driven pipeline that produces Debian build trees
under `build/`, each ready for `dpkg-buildpackage`.

## Current Specs

- [Spec 001 - High-level rules](../catalog/overview/catalog.md)
- [Spec 002 - nvidia-graphics-drivers: Ubuntu driver rebuild for Debian](../catalog/nvidia-graphics-drivers/catalog.md)

## Status

Bootstrap: base folder structure and conventions in place. First subproject
(nvidia-graphics-drivers-595) specified; Ansible role to follow.

## Related

- Spec 002 details the current manual README workflow (download, `dpkg-source
  -x`, strip `dh-modaliases`/`dh_modaliases`, version, build) automated as a
  subproject.
- See [README.md](../../README.md) for the manual workflow this automates.
