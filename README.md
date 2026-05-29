
# Ansible Role: audit

[![CI](https://github.com/guidugli/ansible-role-audit/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-audit/actions/workflows/CI.yml)
[![Release](https://github.com/guidugli/ansible-role-audit/actions/workflows/release.yml/badge.svg)](https://github.com/guidugli/ansible-role-audit/actions/workflows/release.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.audit-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/audit/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

Install and configure Linux auditing with `auditd`, deploy reusable rule templates,
and validate the resulting configuration across modern Debian, Ubuntu LTS, and
Fedora releases.

## Overview

This role is intended for security-focused Linux baselines that want reusable,
composable audit rules instead of one monolithic rules file. It supports package
installation, `auditd.conf` tuning, selective rule deployment into
`/etc/audit/rules.d`, and idempotent rule reloads.

## Features

- Multi-distro support for Debian, Ubuntu LTS, and Fedora.
- Explicit defaults for all public variables.
- Native `meta/argument_specs.yml` validation plus semantic asserts in
  `tasks/assert.yml`.
- Container-aware service handling for Molecule scenarios.
- Shared Molecule structure with generated inventories.
- Generated Galaxy metadata based on the shared Molecule platform matrix.
- Rule template validation to catch typos early.

## Supported platforms

The shared Molecule matrix currently targets:

- Ubuntu 26.04 / 24.04
- Debian 13 / 12
- Fedora 44 / 43

Galaxy metadata is generated from the same source of truth in
`molecule/shared/vars.yml`.

## Role variables

### Core behavior

```yaml
audit_local_events: true
audit_skip_grub_check: false
audit_write_logs: true
```

### `auditd.conf` settings

```yaml
audit_log_file: ''
audit_log_format: enriched
audit_log_group: ''
audit_priority_boost: 4
audit_flush: incremental_async
audit_freq: 50
audit_num_logs: 5
audit_name_format: none
audit_name: ''
audit_max_log_file: 256
audit_max_log_file_action: keep_logs
audit_verify_email: true
audit_action_mail_acct: ''
audit_space_left: 75
audit_space_left_action: email
audit_admin_space_left: 50
audit_admin_space_left_action: suspend
audit_disk_full_action: suspend
audit_disk_error_action: suspend
```

### Remote audit transport

```yaml
audit_tcp_listen_port: 60
audit_tcp_listen_queue: 5
audit_tcp_max_per_addr: 1
audit_use_libwrap: true
audit_tcp_client_ports: '1024-65535'
audit_tcp_client_max_idle: 0
audit_transport: tcp
audit_krb5_principal: ''
audit_krb5_key_file: ''
audit_distribute_network: false
audit_q_depth: 400
audit_overflow_action: syslog
audit_max_restarts: 10
audit_plugin_dir: /etc/audit/plugins.d
```

### Rule deployment

```yaml
force_overwrite_audit: true
audit_sudo_log: "{{ sudo_log | default('/var/log/sudo.log') }}"
audit_rules_files: []
```

## Built-in rule templates

Examples available under `templates/` include:

- `01-init.rules`
- `10-self-audit.rules`
- `20-filters.rules`
- `30-kernel.rules`
- `40-identity.rules`
- `40-login.rules`
- `50-pkg-manager.rules`
- `50-sudoers.rules`
- `55-privileged.rules`
- `60-sshd.rules`
- `80-suspicious.rules`
- `95-32bit-api-exploitation.rules`

The full allow-list is maintained in `vars/main.yml` and is used by
`tasks/assert.yml` to reject unknown template names before deployment.

## How it works

1. Installs audit packages appropriate to the distribution.
2. Optionally validates GRUB kernel parameters on non-container targets.
3. Applies selected `auditd.conf` settings idempotently.
4. Discovers privileged SUID/SGID binaries for `55-privileged.rules`.
5. Deploys selected rule templates into `/etc/audit/rules.d/`.
6. Ensures the immutable finalize rule contains `-e 2`.
7. Reloads rules with `augenrules --load` when configuration changes.

## Usage examples

### Minimal usage

```yaml
---
- name: Configure Linux auditing
  hosts: all
  become: true
  roles:
    - role: guidugli.audit
```

### Selected rule files

```yaml
---
- name: Configure audit rules
  hosts: all
  become: true
  roles:
    - role: guidugli.audit
      vars:
        audit_rules_files:
          - 01-init.rules
          - 10-self-audit.rules
          - 30-kernel.rules
          - 40-identity.rules
          - 50-sudoers.rules
          - 55-privileged.rules
          - 95-32bit-api-exploitation.rules
```

### Remote transport with Kerberos

```yaml
---
- name: Configure remote audit transport
  hosts: audit_servers
  become: true
  roles:
    - role: guidugli.audit
      vars:
        audit_transport: krb5
        audit_krb5_principal: auditd
        audit_krb5_key_file: /etc/audit/audit.key
```

## Design notes

- The role does **not** hardcode `become` inside role tasks. Callers should set
  privilege escalation at the play level.
- Optional string settings use empty strings where “unset” semantics are needed,
  and the tasks only render those directives when the value is non-empty.
- `meta/argument_specs.yml` is the primary input-validation layer; tasks do not
  duplicate `validate_argument_spec`.
- Container scenarios validate file/config behavior and skip assumptions that do
  not make sense for `auditd` runtime behavior inside ordinary containers.

## Molecule testing

This role uses a shared Molecule structure:

```text
molecule/
  shared/
    vars.yml
    converge.yml
    verify.yml
  default/
  systemd/
```

### Run locally

```bash
./scripts/run_local.sh
```

Or run scenarios individually:

```bash
molecule test -s default
molecule test -s systemd
```

## Release workflow

`molecule/shared/vars.yml` is the source of truth for tested platforms. The
release metadata workflow regenerates:

- `molecule/default/inventory/hosts.yml`
- `molecule/systemd/inventory/hosts.yml`
- `meta/main.yml`

Refresh generated artifacts with:

```bash
./scripts/update_release_metadata.sh
```

Prepare a release with:

```bash
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

## Repository structure

```text
defaults/
vars/
tasks/
handlers/
templates/
meta/
molecule/
scripts/
.github/workflows/
```

## License

MIT

## Author

Carlos Guidugli
