# Ansible Role: vector-role

Ansible role for installing and configuring Vector - a high-performance observability data pipeline.

## Description

This role installs Vector from a deb package and ensures the service is running and enabled.

## Requirements

- Ansible >= 2.9
- Debian/Ubuntu based system
- Root or sudo access

## Role Variables

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `vector_version` | `"0.51.1"` | Version of Vector to install |
| `vector_download_url` | `"https://packages.timber.io/vector/{{ vector_version }}/vector_{{ vector_version }}-1_amd64.deb"` | URL to download Vector deb package |
| `vector_deb_path` | `/tmp/vector.deb` | Local path where deb package will be downloaded |
| `vector_service_name` | `vector` | Systemd service name for Vector |

### Example Playbook

```yaml
---
- hosts: vector
  become: true
  roles:
    - vector-role
```

### Customizing Variables

You can override default variables:

```yaml
---
- hosts: vector
  become: true
  vars:
    vector_version: "0.52.0"
  roles:
    - vector-role
```

## Dependencies

None

## Example Playbook

```yaml
---
- name: Install Vector
  hosts: vector
  become: true
  roles:
    - vector-role
```

## License

MIT

## Author Information

This role was created as part of the Ansible course project.
