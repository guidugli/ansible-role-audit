# Ansible Role: `audit`

Install and configure Linux auditing with `auditd`, deploy reusable audit rules, and validate the resulting configuration across modern Debian, Ubuntu LTS, and Fedora releases.

This role is designed to be:
- **multi-distro compatible**
- **security-focused**
- **container-aware** for Molecule testing
- **modular**, so individual audit rule files can be selected as needed

---

## Features

- Installs audit packages appropriate for the target distribution
- Configures `auditd` settings in `/etc/audit/auditd.conf`
- Deploys selected rules into `/etc/audit/rules.d/`
- Generates a final immutable rules file (`99-finalize.rules`)
- Uses `augenrules --load` to regenerate and apply rules
- Skips service-start behavior in containers where `auditd` runtime is not normally usable
- Supports reusable rule templates for:
  - identity changes
  - login/session tracking
  - package management
  - sudo activity
  - kernel/module changes
  - suspicious binaries and reconnaissance activity
  - virtualization/container-related activity

---

## Requirements

- Ansible >= 2.14
- Python available on target hosts
- Root or privilege escalation capability to manage packages and files under `/etc/audit`
- For full runtime behavior on real hosts, the system should not be a container-only environment

> **Important:** On containerized systems, `auditd` usually cannot function the same way it does on a real VM or bare-metal host. This role avoids starting `auditd` in containers, but still supports configuration and file-level validation during Molecule tests.

---

## Supported platforms

This repository is structured to support the latest two:
- Ubuntu LTS releases
- Debian releases
- Fedora releases

The current Molecule matrix is driven from `molecule/shared/vars.yml` and currently includes:

- Ubuntu 26.04 / 24.04
- Debian 13 / 12
- Fedora 44 / 43

---

## Role Variables

## Default Variables (`defaults/main.yml`)

### Core behavior

#### `audit_local_events`
```yaml
audit_local_events: true
```
Include local events in the audit stream.

#### `audit_skip_grub_check`
```yaml
audit_skip_grub_check: false
```
If set to `false`, the role validates that GRUB contains:
- `audit=1`
- `audit_backlog_limit=...`

If those are not present, the role fails and expects a separate GRUB role/process to manage kernel command line configuration.

#### `audit_write_logs`
```yaml
audit_write_logs: true
```
Whether audit events are written to disk.

---

### Audit log file settings

#### `audit_log_file`
```yaml
# audit_log_file: /var/log/audit/audit.log
```
Optional full path to the audit log file.

#### `audit_log_format`
```yaml
audit_log_format: ENRICHED
```
How log records are stored. Common values:
- `ENRICHED`
- `RAW`

#### `audit_log_group`
```yaml
# audit_log_group: root
```
Optional group owner for the audit log file.

#### `audit_priority_boost`
```yaml
# audit_priority_boost: 4
```
Optional non-negative scheduling priority boost for `auditd`.

#### `audit_flush`
```yaml
# audit_flush: INCREMENTAL_ASYNC
```
Flush mode. Supported logical values include:
- `none`
- `incremental`
- `incremental_async`
- `data`
- `sync`

#### `audit_freq`
```yaml
# audit_freq: 50
```
Flush frequency used when `audit_flush` is incremental-based.

#### `audit_num_logs`
```yaml
# audit_num_logs: 5
```
Number of rotated audit log files to keep.

#### `audit_max_log_file`
```yaml
audit_max_log_file: 256
```
Maximum audit log file size in MiB.

#### `audit_max_log_file_action`
```yaml
audit_max_log_file_action: keep_logs
```
Action to take when `audit_max_log_file` is reached.

---

### Node identity / mail / low-space handling

#### `audit_name_format`
```yaml
# audit_name_format: NONE
```
How node identity is added to events:
- `none`
- `hostname`
- `fqd`
- `numeric`
- `user`

#### `audit_name`
```yaml
# audit_name: mydomain
```
Required when `audit_name_format` is `user`.

#### `audit_verify_email`
```yaml
# audit_verify_email: true
```
Whether to verify the domain from `audit_action_mail_acct`.

#### `audit_action_mail_acct`
```yaml
# audit_action_mail_acct: root
```
Email address or alias used when auditd sends alerts.

#### `audit_space_left`
```yaml
# audit_space_left: 75
```
Remaining filesystem threshold before action is taken.
Can be either:
- integer MiB
- percentage (for example `5%`)

#### `audit_space_left_action`
```yaml
# audit_space_left_action: email
```
Action when free space starts running low.

#### `audit_admin_space_left`
```yaml
# audit_admin_space_left: 50
```
Last-chance low-space threshold in MiB.

#### `audit_admin_space_left_action`
```yaml
audit_admin_space_left_action: suspend
```
Action at the admin threshold.

#### `audit_disk_full_action`
```yaml
audit_disk_full_action: SUSPEND
```
Action when the audit filesystem becomes full.

#### `audit_disk_error_action`
```yaml
audit_disk_error_action: SUSPEND
```
Action when a write or rotation error occurs.

---

### Remote audit transport settings

#### `audit_tcp_listen_port`
```yaml
# audit_tcp_listen_port: 60
```
TCP port for receiving remote audit events.

#### `audit_tcp_listen_queue`
```yaml
# audit_tcp_listen_queue: 5
```
Pending remote connection backlog.

#### `audit_tcp_max_per_addr`
```yaml
# audit_tcp_max_per_addr: 1
```
Maximum concurrent remote audit connections per source IP.

#### `audit_use_libwrap`
```yaml
# audit_use_libwrap: true
```
Whether `tcp_wrappers` should be used.

#### `audit_tcp_client_ports`
```yaml
# audit_tcp_client_ports: 1024-65535
```
Allowed incoming client source ports.

#### `audit_tcp_client_max_idle`
```yaml
# audit_tcp_client_max_idle: 0
```
Maximum idle time for remote clients.

#### `audit_transport`
```yaml
# audit_transport: TCP
```
Transport for remote audit communication:
- `TCP`
- `KRB5`

#### `audit_krb5_principal`
```yaml
# audit_krb5_principal: auditd
```
Kerberos principal used with `KRB5` transport.

#### `audit_krb5_key_file`
```yaml
# audit_krb5_key_file: /etc/audit/audit.key
```
Kerberos key file path.

#### `audit_distribute_network`
```yaml
# audit_distribute_network: false
```
Whether remote-originating events are distributed to the dispatcher.

#### `audit_q_depth`
```yaml
# audit_q_depth: 400
```
Internal dispatcher queue depth.

#### `audit_overflow_action`
```yaml
# audit_overflow_action: SYSLOG
```
Action when the dispatcher queue overflows.

#### `audit_max_restarts`
```yaml
# audit_max_restarts: 10
```
Maximum plugin restarts after crashes.

#### `audit_plugin_dir`
```yaml
# audit_plugin_dir: /etc/audit/plugins.d
```
Path to plugin configuration files.

---

### Rule deployment behavior

#### `force_overwrite_audit`
```yaml
force_overwrite_audit: true
```
Whether deployed audit rule files should be overwritten when their content differs.

#### `audit_sudo_log`
```yaml
audit_sudo_log: "{{ sudo_log | default('/var/log/sudo.log') }}"
```
Used by the `50-sudoers.rules` template so the sudo log path can align with a sudo role if one is also used.

#### `audit_rules_files`
```yaml
audit_rules_files: []
```
List of rule template files to deploy into `/etc/audit/rules.d/`.

The role intentionally provides many small reusable rule files instead of one monolithic ruleset.

Examples available in `templates/` include:
- `01-init.rules`
- `10-self-audit.rules`
- `20-filters.rules`
- `30-kernel.rules`
- `40-identity.rules`
- `40-login.rules`
- `40-mount.rules`
- `40-swap.rules`
- `40-time.rules`
- `50-cron.rules`
- `50-dac.rules`
- `50-hostname.rules`
- `50-network.rules`
- `50-pkg-manager.rules`
- `50-remote-shell.rules`
- `50-sudoers.rules`
- `55-privileged.rules`
- `60-pam.rules`
- `60-sshd.rules`
- `60-systemd.rules`
- `70-access.rules`
- `70-mac-policy.rules`
- `70-sessions.rules`
- `70-shell-profiles.rules`
- `80-privilege-abuse.rules`
- `80-reconnaissance.rules`
- `80-socket-creation.rules`
- `80-suspicious.rules`
- `80-suspicious-shells.rules`
- `80-virtualization.rules`
- `90-root-exec.rules`
- `95-32bit-api-exploitation.rules`

---

## Internal Variables (`vars/main.yml`)

These are internal role variables and normally should not need user overrides.

### `audit_packages`
Distribution-aware package list derived from `_audit_packages`.

### `audit_config_path`
```yaml
audit_config_path: /etc/audit/auditd.conf
```

### `audit_rulesd_path`
```yaml
audit_rulesd_path: /etc/audit/rules.d
```

### `_container_types`
Container/virtualization types treated as container-like in service logic.

### `_audit_vars`
Mapping used to translate role variable names into auditd.conf parameter names.

---

## Validation model

This role uses a two-layer validation approach:

1. **`meta/argument_specs.yml`**
   - validates supported inputs
   - checks types and basic allowed values

2. **`tasks/assert.yml`**
   - validates cross-field semantics that are better expressed in asserts
   - examples:
     - `audit_flush` may require `audit_freq`
     - `audit_name_format: user` requires `audit_name`
     - `audit_transport: krb5` requires Kerberos settings

---

## Example Playbook

### Minimal example

```yaml
- name: Configure Linux auditing
  hosts: all
  become: true

  roles:
    - role: guidugli.audit
```

### Example with selected rule files

```yaml
- name: Configure Linux auditing with selected rules
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

### Example enabling remote audit transport settings

```yaml
- name: Configure remote audit transport
  hosts: audit_servers
  become: true

  roles:
    - role: guidugli.audit
      vars:
        audit_transport: TCP
        audit_tcp_listen_port: 60
        audit_tcp_listen_queue: 5
        audit_tcp_max_per_addr: 2
```

---

## What the role configures

### 1. Audit packages
Installs the correct packages for the target platform.

### 2. `auditd.conf`
Writes selected configuration values into:

```text
/etc/audit/auditd.conf
```

### 3. Rule files
Deploys selected templates into:

```text
/etc/audit/rules.d/
```

### 4. Immutable final rule
Ensures:

```text
/etc/audit/rules.d/99-finalize.rules
```

contains:

```text
-e 2
```

### 5. Rule reload
Uses:

```bash
augenrules --load
```

to regenerate and apply rules when changes occur.

---

## Testing

This role uses **Molecule + Podman** and follows the same shared/default/systemd scenario structure used in the other modernized roles.

### Scenarios
- `default`
- `systemd`

### Run locally

```bash
./scripts/run_local.sh
```

or individually:

```bash
molecule test -s default
molecule test -s systemd
```

### Verify strategy
The shared verify playbook is audit-specific and focuses on:
- `auditctl` binary presence
- config file existence
- rule directory existence
- finalize rule presence and content
- deployed rule file permissions
- runtime checks only where meaningful

---

## Release metadata workflow

Like the updated sudo/chrony-style roles, this role uses generated metadata based on the shared Molecule matrix.

### Source of truth

```text
molecule/shared/vars.yml
```

This drives:
- tested platforms
- generated inventories
- generated `meta/main.yml`

### Refresh metadata

```bash
./scripts/update_release_metadata.sh
```

### Prepare a release

```bash
./scripts/release.sh --version v1.2.0 --message "Release v1.2.0"
```

---

## Relevant project structure

```text
defaults/
  main.yml

vars/
  main.yml

tasks/
  main.yml
  assert.yml
  audit.yml

handlers/
  main.yml

templates/
  *.rules
  meta_main.yml.j2

meta/
  main.yml
  argument_specs.yml

molecule/
  shared/
  default/
  systemd/
```

---

## Design notes

- This role is intentionally **modular** rather than shipping a single opinionated monolithic rules file.
- Containers are treated as a **test/configuration target**, not a full runtime replacement for host audit behavior.
- GRUB/kernel command line validation is intentionally separated from full bootloader management.
- The role is suitable for building larger compliance/hardening baselines where audit rules are selected per environment.

---

## License

MIT

---

## Author

Carlos Guidugli
