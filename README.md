[![CI](https://github.com/guidugli/ansible-role-openssh/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-openssh/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-openssh?sort=semver)](https://github.com/guidugli/ansible-role-openssh/tags)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.openssh-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/openssh/)
[![License](https://img.shields.io/github/license/guidugli/ansible-role-openssh)](https://github.com/guidugli/ansible-role-openssh/blob/main/LICENSE)

# Ansible Role: openssh

An Ansible role to install and configure OpenSSH on RHEL/CentOS, Fedora, and Debian/Ubuntu targets. The default values are based on CIS-style hardening guidance while preserving the original variable intent.

## Requirements

- Ansible Core 2.17+
- `containers.podman` collection for Molecule scenarios
- `community.general` collection for SELinux port management (`seport`)

## Variables

The available variables are listed below together with the meaning preserved from the original role documentation.

### Configuration file and service port

- `openssh_config_file` (default: `/etc/ssh/sshd_config`): configuration file to update. The original role notes that more modern OpenSSH versions may prefer storing configuration in `sshd_config.d`, but this role still targets the main SSH daemon configuration file by default.
- `openssh_port` (default: `22`): SSH service port number. The role can change the port, but success can still depend on firewall, SELinux, and connectivity conditions.

### Logging, sessions, and authentication

- `openssh_log_level` (default: `INFO`): set the SSH daemon logging level.
- `openssh_x11forwarding` (default: `false`): enable or disable X11 forwarding.
- `openssh_max_auth_tries` (default: `4`): maximum number of authentication attempts permitted per connection.
- `openssh_client_alive_interval` (default: `300`): timeout interval in seconds after which sshd sends a message through the encrypted channel to request a response from the client.
- `openssh_client_alive_count_max` (default: `0`): number of unanswered client alive messages allowed before disconnecting the client.
- `openssh_login_grace_time` (default: `60`): time allowed for a user to log in before the server disconnects the session.
- `openssh_password_authentication` (default: `true`): enable or disable password authentication.
- `openssh_use_pam` (default: `true`): enable or disable PAM integration. The original README notes that PAM challenge-response can overlap with password authentication.
- `openssh_permit_empty_passwords` (default: `false`): allow or deny accounts with empty passwords.
- `openssh_permit_root_login` (default: `'no'`): root login policy. Supported values are `yes`, `no`, `without-password`, `prohibit-password`, and `forced-commands-only`.
- `openssh_permit_user_environment` (default: `false`): control whether `~/.ssh/environment` and `environment=` options in `authorized_keys` are processed.
- `openssh_ignore_rhosts` (default: `true`): prevent the use of `.rhosts` and `.shosts` files.
- `openssh_host_based_authentication` (default: `false`): enable or disable host-based authentication.

### Access control and forwarding

- `openssh_banner` (default: `/etc/issue.net`): banner file sent to the remote user before authentication. The original role notes that `none` disables the banner.
- `openssh_allow_tcp_forwarding` (default: `false`): enable or disable TCP forwarding.
- `openssh_max_sessions` (default: `4`): maximum number of open sessions permitted per network connection.
- `openssh_max_startups` (default: `'10:30:60'`): maximum number of concurrent unauthenticated connections allowed by sshd, using the `start:rate:full` format documented by the original role.
- `openssh_use_dns` (default: `true`): control whether sshd resolves the remote host name and verifies reverse mapping.
- `openssh_permit_tunnel` (default: `false`): control whether tunnel device forwarding is allowed.

### Allow and deny lists

- `openssh_allow_users` (default: `[]`): list of user patterns allowed to log in.
- `openssh_allow_groups` (default: `[]`): list of groups allowed to log in.
- `openssh_deny_users` (default: `[]`): list of user patterns denied login.
- `openssh_deny_groups` (default: `[]`): list of groups denied login.

### Cryptographic settings

The following variables are not applied on systems where a compatible crypto-policy package is installed, as documented in the original role.

- `openssh_macs`: list of allowed MAC algorithms.
- `openssh_kex`: list of allowed key exchange algorithms.
- `openssh_ciphers`: list of allowed cipher algorithms.

### OS-dependent internal variables

The original role documented these as values that typically do not need to be changed on target systems.

- `host_keys_group`: group that owns host keys, derived from the target operating system.
- `host_keys_mode`: permissions applied to host keys, derived from the target operating system.
- `openssh_service_name`: service name for the SSH daemon on the target operating system.

## Example playbook

```yaml
---
- name: Configure OpenSSH
  hosts: servers
  become: true
  vars:
    openssh_port: 10999
    openssh_allow_users:
      - user1
      - user2
  roles:
    - role: guidugli.openssh
```

## Molecule testing

Run the fast container validation scenario:

```bash
molecule test -s default
```

Run the systemd-oriented validation scenario:

```bash
molecule test -s systemd
```

## Execution notes

- **Privilege model:** the role does not define `become`, `become_user`, or `become_method`. Callers should set `become: true` when targeting hosts that require package management, service changes, SELinux changes, or updates under `/etc/ssh`.
- **Container behavior:** the default Molecule scenario assumes container-root execution and keeps privilege handling outside the role.
- **Systemd behavior:** the role manages the SSH service, so service-related validation is more representative in the `systemd` Molecule scenario.
