Ansible Role: audit
=========

An Ansible Role that install and configure auditing on RHEL/CentOS, Fedora and Debian/Ubuntu.

Requirements
------------

Operating system running on bare metal or on a hypervisor virtualization. On conteinerized systems, only one auditd process is able to run, usually on the host system. This role will not attempt to start auditd daemon on containers.

Role Variables
--------------

**Available variables are listed below, along with default values (see `defaults/main.yml`):**

    audit_log_file_size: 256

Maximum size of the audit log file, in megabytes

    force_overwrite_audit: true

If audit file already exists, force overwrite? It will overwrite only if target file contents are different from source.

    audit_sudo_log: "{{ sudo_log | default('/var/log/sudo.log') }}"

If sudo role is defined, get value from sudo_log variable otherwise use default: /var/log/sudo.log
This is to prevent users from having to specify the same information multiple times. But sudo role is not a dependency for this role.

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    audit_packages: ['audit', 'audit-libs']

This variable is defined in vars/main.yml and it is set according to the Linux distribution. Users don't need to change this variable for the target systems of this role.

    audit_config_path: /etc/audit/auditd.conf

This variable is defined in vars/main.yml and controls where the auditd.conf file is located.

    audit_rulesd_path: /etc/audit/rules.d

This variable is defined in vars/main.yml and controls where the rules.d directory is located.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      vars_files:
        - vars/main.yml
      roles:
         - { role: guidugli.audit }
      
License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
