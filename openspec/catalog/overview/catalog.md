# Specifications Catalog

## Spec 001 - Corpos: Adapting non-Debian packages for Debian with Ansible

[spec](../spec/overview/spec.md)

| Field | Value |
| ----- | ----- |
| Status | Proposed |
| Area | Build & packaging conventions |
| Spec  | 001 |

Establishes the base high-level rules of the project:

- All build data lives under `build/`.
- Each `dpkg-buildpackage` run targets its own folder under `build/`.
- Ansible is the single entry point for the entire adaptation workflow.
- Each adapted package has its own role and Spec.
- The pipeline is reproducible and `build/` is disposable.
