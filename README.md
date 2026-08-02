[![CI](https://github.com/guidugli/ansible-role-audit/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-audit/actions/workflows/CI.yml)
[![Release](https://img.shields.io/github/v/tag/guidugli/ansible-role-audit?label=release)](https://github.com/guidugli/ansible-role-audit/tags)
[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-guidugli.audit-blue)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/audit/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

# Ansible Role: audit

Installs and configures Linux auditd, manages selected reusable audit rule templates, and validates the resulting configuration across supported Debian, Ubuntu, and Fedora targets.

## Requirements

- Ansible Core 2.14 or newer, as declared by role metadata.
- A supported Linux target with the distribution audit packages.
- Root-level access for package, service, `/etc/audit`, and audit-rule operations. Supply privilege externally.
- `containers.podman` 1.10.0 or newer for the bundled Molecule scenarios.

## Features

- Distribution-aware audit package installation.
- Idempotent `auditd.conf` management.
- Allow-listed, selectable audit rule deployment.
- GRUB audit parameter validation on applicable non-container systems.
- Container-aware service and reload behavior.
- Argument-spec validation for all public role inputs.
- Shared Molecule converge and verification plays.

## Supported platforms

The shared Molecule matrix covers Ubuntu 26.04 and 24.04, Debian 13 and 12, and Fedora 44 and 43. Galaxy platform metadata is generated from that matrix.

## Variables

All public variables are defined in `defaults/main.yml` and validated by `meta/argument_specs.yml`.

### Core behavior

| Variable | Type | Default | Description |
|---|---|---|---|
| `audit_local_events` | bool | `true` | Include local events in the audit stream. |
| `audit_skip_grub_check` | bool | `false` | Skip checks for `audit=1` and `audit_backlog_limit` in GRUB configuration. |
| `audit_write_logs` | bool | `true` | Write audit events to disk. |

### Audit log and rotation

| Variable | Type | Default | Description |
|---|---|---|---|
| `audit_log_file` | string | `''` | Optional audit log path. Empty leaves the existing directive unchanged. |
| `audit_log_format` | string | `enriched` | Audit log format, `raw` or `enriched`. |
| `audit_log_group` | string | `''` | Optional group allowed to read audit logs. |
| `audit_priority_boost` | int | `4` | Audit daemon priority boost. |
| `audit_flush` | string | `incremental_async` | Log flush strategy. |
| `audit_freq` | int | `50` | Records between flushes for incremental modes. |
| `audit_num_logs` | int | `5` | Number of rotated logs retained. |
| `audit_name_format` | string | `none` | Host-name format included in events. |
| `audit_name` | string | `''` | Explicit host name when `audit_name_format` is `user`. |
| `audit_max_log_file` | int | `256` | Maximum audit log size in MiB. |
| `audit_max_log_file_action` | string | `keep_logs` | Action when the maximum log size is reached. |
| `audit_verify_email` | bool | `true` | Verify the configured notification account. |
| `audit_action_mail_acct` | string | `''` | Optional notification account. |
| `audit_space_left` | raw | `75` | Free-space threshold in MiB or a percentage string. |
| `audit_space_left_action` | string | `email` | Action at the free-space threshold. |
| `audit_admin_space_left` | int | `50` | Administrative free-space threshold. |
| `audit_admin_space_left_action` | string | `suspend` | Action at the administrative threshold. |
| `audit_disk_full_action` | string | `suspend` | Action when the filesystem is full. |
| `audit_disk_error_action` | string | `suspend` | Action on audit-log filesystem errors. |

### Remote transport and plugins

| Variable | Type | Default | Description |
|---|---|---|---|
| `audit_tcp_listen_port` | int | `60` | TCP listener port. |
| `audit_tcp_listen_queue` | int | `5` | Pending TCP connection queue length. |
| `audit_tcp_max_per_addr` | int | `1` | Maximum connections per source address. |
| `audit_use_libwrap` | bool | `true` | Enable libwrap access controls. |
| `audit_tcp_client_ports` | string | `'1024-65535'` | Allowed client source-port range. |
| `audit_tcp_client_max_idle` | int | `0` | Client idle timeout. |
| `audit_transport` | string | `tcp` | Remote transport, `tcp` or `krb5`. |
| `audit_krb5_principal` | string | `''` | Kerberos principal required by `krb5` transport. |
| `audit_krb5_key_file` | string | `''` | Kerberos key file required by `krb5` transport. |
| `audit_distribute_network` | bool | `false` | Distribute network-originated events. |
| `audit_q_depth` | int | `400` | Event queue depth. |
| `audit_overflow_action` | string | `syslog` | Action when the queue overflows. |
| `audit_max_restarts` | int | `10` | Maximum plugin restart attempts. |
| `audit_plugin_dir` | string | `/etc/audit/plugins.d` | Audit plugin configuration directory. |

### Rule deployment

| Variable | Type | Default | Description |
|---|---|---|---|
| `force_overwrite_audit` | bool | `true` | Replace managed rule files when content differs. |
| `audit_sudo_log` | string | `/var/log/sudo.log` | Sudo log path referenced by `50-sudoers.rules`. |
| `audit_rules_files` | list(string) | See baseline below | Ordered allow-listed templates deployed into `/etc/audit/rules.d`. |

Default cross-platform baseline:

```yaml
audit_rules_files:
  - 40-identity.rules
  - 40-login.rules
  - 50-network.rules
  - 50-sudoers.rules
  - 55-privileged.rules
  - 60-pam.rules
  - 70-mac-policy.rules
  - 70-sessions.rules
```

This default closes the empty-rule-set gap and is intentionally conservative. It is not a complete CIS Debian 13 or RHEL 10 profile. Distribution-specific syscall, architecture, retention, and runtime reconciliation controls still require dedicated profiles and host-level compliance validation.

### Audit file-access controls

| Variable | Type | Default | Description |
|---|---|---|---|
| `audit_manage_file_access` | bool | `false` | Manage ownership and permissions for the audit log directory, existing audit logs, audit configuration files, and installed audit tools. |
| `audit_log_directory_mode` | string | `'0750'` | Exact mode applied to the effective audit log directory. |
| `audit_log_file_mode` | string | `'0640'` | Exact mode applied to existing regular files in the effective audit log directory. |
| `audit_log_file_owner` | string | `root` | Owner applied to existing audit log files. |
| `audit_log_file_group` | string | `root` | Group applied to the audit log directory and existing audit log files. |
| `audit_config_file_mode` | string | `'0640'` | Exact mode applied to `*.conf` and `*.rules` files below `/etc/audit`. |
| `audit_config_file_owner` | string | `root` | Owner applied to audit configuration and rule files. |
| `audit_config_file_group` | string | `root` | Group applied to audit configuration and rule files. |
| `audit_tool_owner` | string | `root` | Owner applied to configured audit tools that exist. |
| `audit_tool_group` | string | `root` | Group applied to configured audit tools that exist. |
| `audit_tool_mode` | string | `'0755'` | Exact mode applied to configured audit tools that exist. |
| `audit_tools` | list(string) | See defaults | Audit tool paths inspected and managed when present. |

## Example playbook

```yaml
---
- name: Configure Linux auditing
  hosts: all
  become: true
  roles:
    - role: guidugli.audit
      vars:
        audit_rules_files:
          - 01-init.rules
          - 40-identity.rules
          - 50-sudoers.rules
          - 55-privileged.rules
```

## Molecule testing instructions

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
ansible-galaxy collection install -r requirements.yml
molecule test -s default
molecule test -s systemd
```

A convenience wrapper is also available as `./scripts/run_local.sh`.

## Execution notes

- **Privilege model:** role tasks never set privilege escalation. Use `become: true` at play, inventory, or automation-controller level for real hosts.
- **Container behavior:** Molecule containers execute as root. Service start/reload and GRUB checks are skipped for recognized container connections or virtualization types.
- **Systemd behavior:** auditd service tasks retain existing runtime guards. The systemd scenario supplies a systemd-capable container for service verification.
- **Immutable rules:** the role writes `99-finalize.rules` with `-e 2`. Review the operational impact before deployment because changing loaded rules can require a reboot.

## Release workflow

Refresh generated metadata and inventories, then prepare a release:

```bash
./scripts/update_release_metadata.sh
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

## License

MIT

## Author

Carlos Guidugli
