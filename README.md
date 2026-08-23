[![CI](https://github.com/guidugli/ansible-role-openssh/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-openssh/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-openssh?label=release)](https://github.com/guidugli/ansible-role-openssh/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.openssh-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/openssh/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-openssh)](LICENSE)

# Ansible Role: openssh

Installs the OpenSSH server, applies hardening-oriented daemon settings, manages host-key permissions, and keeps SSH service handling safe across systemd and container execution contexts.

## Requirements

- Ansible Core 2.17 or newer.
- A supported Linux target with Python available after bootstrap.
- External privilege escalation for package, service, SELinux, and `/etc/ssh` changes.
- `community.general` 10.0.0 or newer for SELinux port management.
- `containers.podman` 1.10.0 or newer for Molecule.

## Features

- Installs `openssh-server` with native package modules.
- Configures authentication, session, forwarding, access-control, and cryptographic directives.
- Validates each configuration edit with `sshd -t` before applying it.
- Preserves distribution-specific SSH service names and host-key ownership modes.
- Avoids overriding Red Hat crypto policy when `crypto-policies` is installed.
- Manages a custom SELinux SSH port when SELinux support is available and enabled.
- Uses handlers for restart operations and guards service actions to systemd hosts.
- Includes shared converge and verification playbooks for default and systemd Molecule scenarios.

## Supported platforms

Role metadata and the shared Molecule matrix cover Fedora, Ubuntu, and Debian. The bundled matrix targets Ubuntu 26.04 and 24.04, Debian 13 and 12, and Fedora 44 and 43.

## Variables

All public inputs are declared in `defaults/main.yml` and validated by `meta/argument_specs.yml`.

### Paths and service endpoint

| Variable | Type | Default | Description |
|---|---|---|---|
| `openssh_config_file` | string | `/etc/ssh/sshd_config` | SSH daemon configuration file managed by the role. |
| `openssh_port` | integer | `22` | Listening port written to `sshd_config`; firewall and connection routing remain caller responsibilities. |
| `openssh_banner` | string | `/etc/issue.net` | Existing pre-authentication banner file. The role fails when the configured file is absent. |

### Authentication, sessions, and forwarding

| Variable | Type | Default | Description |
|---|---|---|---|
| `openssh_log_level` | string | `INFO` | SSH daemon log level. |
| `openssh_x11forwarding` | boolean | `false` | Enables X11 forwarding. |
| `openssh_max_auth_tries` | integer | `4` | Maximum authentication attempts per connection. |
| `openssh_client_alive_interval` | integer | `300` | Seconds between encrypted client-alive probes. |
| `openssh_client_alive_count_max` | integer | `0` | Unanswered client-alive probes allowed before disconnect. |
| `openssh_login_grace_time` | integer | `60` | Seconds allowed to authenticate. |
| `openssh_allow_tcp_forwarding` | boolean | `false` | Enables TCP forwarding. |
| `openssh_max_sessions` | integer | `4` | Maximum sessions per network connection. |
| `openssh_ignore_rhosts` | boolean | `true` | Ignores `.rhosts` and `.shosts`. |
| `openssh_host_based_authentication` | boolean | `false` | Enables host-based authentication. |
| `openssh_permit_root_login` | string | `no` | Root login policy: `yes`, `no`, `without-password`, `prohibit-password`, or `forced-commands-only`. |
| `openssh_permit_empty_passwords` | boolean | `false` | Permits accounts with empty passwords. |
| `openssh_permit_user_environment` | boolean | `false` | Permits user environment processing. |
| `openssh_use_pam` | boolean | `true` | Enables PAM integration. |
| `openssh_max_startups` | string | `10:30:60` | Unauthenticated connection limit in `start:rate:full` format. |
| `openssh_use_dns` | boolean | `true` | Enables remote hostname lookup and reverse mapping checks. |
| `openssh_password_authentication` | boolean | `true` | Enables password authentication. |
| `openssh_permit_tunnel` | boolean | `false` | Enables tunnel device forwarding. |

### Access-control lists

| Variable | Type | Default | Description |
|---|---|---|---|
| `openssh_allow_users` | list of strings | `[]` | User patterns allowed to log in. Empty removes the directive. |
| `openssh_allow_groups` | list of strings | `[]` | Group patterns allowed to log in. Empty removes the directive. |
| `openssh_deny_users` | list of strings | `[]` | User patterns denied login. Empty removes the directive. |
| `openssh_deny_groups` | list of strings | `[]` | Group patterns denied login. Empty removes the directive. |

### Cryptographic policy

| Variable | Type | Default | Description |
|---|---|---|---|
| `openssh_macs` | list of strings | Hardened SHA-2 MAC list | MAC algorithms written when a system crypto-policy package is not active. |
| `openssh_kex` | list of strings | Curve25519, ECDH, and DH SHA-2 list | Key-exchange algorithms written when system crypto policy is not active. |
| `openssh_ciphers` | list of strings | ChaCha20, AES-GCM, and AES-CTR list | Ciphers written when system crypto policy is not active. |

`host_keys_group`, `host_keys_mode`, and `openssh_service_name` are internal, OS-derived variables in `vars/main.yml`, not public role inputs.

## Example playbook

```yaml
---
- name: Configure OpenSSH
  hosts: servers
  become: true
  roles:
    - role: guidugli.openssh
      vars:
        openssh_port: 10999
        openssh_password_authentication: false
        openssh_allow_users:
          - automation
          - operator
```

Changing the daemon port can interrupt future connections. Update inventory, firewall policy, SELinux policy, and recovery access as appropriate before rollout.

## Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

The default scenario validates configuration in regular Podman containers. The systemd scenario validates service behavior in systemd-capable privileged containers.

## Execution notes

- **Privilege model:** the role never declares `become`, `become_user`, or `become_method`. Use `become: true` at the play, inventory, or automation-controller level on real hosts.
- **Privileged operations:** package installation, service management, SELinux port changes, host-key permission changes, and writes under `/etc` require root-equivalent permissions.
- **Container behavior:** shared Molecule converge runs as container root with `become: false`. Service operations are skipped when the detected service manager is not systemd.
- **OpenSSH validation prerequisites:** the role creates `/run/sshd` and generates only missing system host keys before running `sshd -t`, including in minimal containers without an active init system.
- **Systemd behavior:** service start and restart actions run only when `ansible_facts['service_mgr'] == 'systemd'`. The systemd scenario provides service-manager coverage.
- **Connection behavior:** the role temporarily probes SSH port reachability and restores the original inventory connection port after configuration.

## Release workflow

Generated metadata and inventories remain controlled by the repository generator scripts:

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

Run lint and both Molecule scenarios before tagging. The release workflow imports tags matching `v*` into Galaxy.

## License

MIT

## Author

Carlos Guidugli
