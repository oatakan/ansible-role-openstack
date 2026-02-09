# ansible-role-openstack

Ansible role to provision and deprovision OpenStack compute instances using `openstack.cloud.server` (direct Nova server lifecycle).

This repository is intentionally low-profile and primarily exists for internal/lab automation and Molecule-driven integration testing.

## What it does

- Validates requested node definitions (preflight: networks/images/flavors/keypairs).
- Creates one or more servers (sync or async paths).
- Optionally waits for an IP address and (for Windows) can wait for WinRM connectivity.
- Deletes servers during deprovision.

## Requirements

- `ansible-core` and the `openstack.cloud` collection.
- OpenStack credentials available via either:
  - environment variables (OpenRC-style `OS_*`), and/or
  - `clouds.yaml` (`OS_CLIENT_CONFIG_FILE` + `OS_CLOUD`).

## Key variables

- `role_action`: `provision` or `deprovision`.
- `nodes`: list of node definitions.
- `openstack_validate_certs`: defaults to `true` (secure by default).
- `config_file`: optional clouds.yaml path (defaults from `OS_CLIENT_CONFIG_FILE`). The role validates it exists when set.

Notes:

- If Glance has multiple images with the same name, prefer passing an image **ID**.
- Some volume/ISO workflows rely on OpenRC-style `OS_*` variables being present.

## Node definitions

Both this role and `ansible-role-os-stack` use the same high-level contract: `nodes` is a list of dictionaries describing desired instances.

Validated (fail-fast) fields:

- Required per node: `name`, `flavor`, `key_name`, `nics` (non-empty list)
- Required per nic: `net-name`
- Optional: `os_type` (`linux` or `windows`)
- Constraint: a node must not set both `image` and `volume_iso_template`

Common useful fields:

- `image`: Glance image name or ID (recommended: ID)
- `os_type`: affects defaults (e.g., Windows userdata + WinRM waiting)
- `user_data`: cloud-init/userdata content
- `auto_ip`: whether OpenStack should auto-assign a floating IP (default: `true` when no explicit floating IPs are provided)
- `floating_ips`: list of floating IPs to attach (if provided, `auto_ip` is omitted)
- `ansible_port`: overrides the connectivity check port (default: 22 for Linux, 5986 for Windows)

OpenStack-server specific fields (advanced):

- `volumes`: additional volume names/IDs to attach
- `boot_from_volume`, `terminate_volume`, `volume_size`: passed through to `openstack.cloud.server`
- `volume_iso_template`: triggers the ISO/volume workflow implemented in this role

## Usage (minimal example)

```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: oatakan.openstack
      vars:
        role_action: provision
        nodes:
          - name: example-1
            os_type: linux
            image: cirros 0.6
            flavor: m1.small
            key_name: my-key
            nics:
              - net-name: private_network
            auto_ip: true
```

## Molecule

This repo uses localhost-only Molecule scenarios that talk to a real OpenStack.

- Default: `./scripts/molecule.sh test -s default`
- Extended: `./scripts/molecule.sh test -s extended`

The wrapper script:

- sources a local `.env` (ignored by git), and
- can optionally source an OpenRC file via `OS_ENV_FILE`.

See [molecule/README.md](molecule/README.md) for environment variables used by the scenarios.

## CI

`ansible-lint` is run via GitHub Actions using `.github/workflows/ansible-lint.yml`.

## Security / secrets

- Do not commit `.env`, OpenRC files, `clouds.yaml`, or any local artifacts; `.gitignore` is configured to ignore these.
- Avoid embedding secrets inside `nodes` (especially in any `user_data` fields): validation errors and failed runs may echo portions of node definitions to logs.
- For Windows userdata, providing an administrator password in userdata is inherently sensitive and may be persisted/logged by your OpenStack control plane.
