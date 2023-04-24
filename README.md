Ansible Role: audit
=========

An Ansible Role that install and configure auditing on RHEL/CentOS, Fedora and Debian/Ubuntu. 

Default rules are based on CIS RedHat/Ubuntu/Debian benchmark and also on this github repository https://github.com/Neo23x0/auditd[https://github.com/Neo23x0/auditd]

Disclaimer: I am no security expert and by no means you should use this role without first learning how audit works and the rules configured by this role. The security of your servers is YOUR responsibility!

Requirements
------------

Operating system running on bare metal or on a hypervisor virtualization. On conteinerized systems, only one auditd process is able to run, usually on the host system. This role will not attempt to start auditd daemon on containers.

Role Variables
--------------

**Available variables are listed below, along with default values (see `defaults/main.yml`):**

    audit_local_events: true

This yes/no keyword specifies whether or not to include local events. Normally you want local events so the default value is yes. Cases where you would set this to no is when you want to aggregate events only from the network.

    #audit_log_file: /var/log/audit/audit.log

This keyword specifies the full path name to the log file where audit records will be stored. It must be a regular file.

    audit_write_logs: true

This yes/no keyword determines whether or not to write logs to the disk.  Normally you want this so the default is true.

    audit_log_format: ENRICHED

The log format describes how the information should be stored on disk. There are 2 options: raw and enriched.

    audit_log_group: root

This keyword specifies the group that is applied to the log file's permissions. The default is root. The group name can be either numeric or spelled out.

    audit_priority_boost: 4

This is a non-negative number that tells the audit daemon how much of a priority boost it should take. The default is 4. No change is 0.

    audit_flush: INCREMENTAL_ASYNC

Valid values are none, incremental, incremental_async, data,  and sync.

    audit_freq: 50

This is a non-negative number that tells the audit daemon how many records to write before issuing an explicit flush to disk command. This value is only valid when the flush keyword is set to incremental or incremental_async.

    audit_num_logs: 5

This keyword specifies the number of log files to keep if rotate is given as the max_log_file_action.  If the number is < 2, logs are not rotated. This number must be 999 or less.

    audit_name_format: NONE

This option controls how computer node names are inserted into the audit event stream. It has the following choices: none, hostname, fqd, numeric, and user.  None means that no computer name is inserted into the audit event.

    #audit_name: mydomain

This is the admin defined string that identifies the machine if user is given as the name_format option.

    audit_max_log_file: 256

This keyword specifies the maximum file size in megabytes. When this limit is reached, it will trigger a configurable action.

    audit_max_log_file_action: keep_logs

This parameter tells the system what action to take when the system has detected that the max file size limit has been reached. Valid values are ignore, syslog, suspend, rotate and keep_logs.

    audit_verify_email: true

This option determines if the email address given in action_mail_acct is checked to see if the domain name can be resolved. This option must be given before action_mail_acct or the default value of yes will be used.

    audit_action_mail_acct: root

This option should contain a valid email address or alias. The default address is root. If the email address is not local to the machine, you must make sure you have email properly configured on your machine and network. Also, this option requires that /usr/lib/sendmail exists on the machine.

    audit_space_left: 75

If the free space in the filesystem containing log_file drops below this value, the audit daemon takes the action specified by space_left_action.  If the value of space_left is specified as a whole number, it is interpreted as an absolute size in megabytes (MiB).  If the value is specified as a number between 1 and 99 followed by a percentage sign (e.g., 5%), the audit daemon calculates the absolute size in megabytes based on the size of the filesystem containing log_file.

    audit_space_left_action: email

This parameter tells the system what action to take when the system has detected that it is starting to get low on disk space.  Valid values are ignore, syslog, rotate, email, exec, suspend, single, and halt.

    audit_admin_space_left: 50

This is a numeric value in megabytes that tells the audit daemon when to perform a configurable action because the system is running low on disk space. This should be considered the last chance to do something before running out of disk space.

    audit_admin_space_left_action: suspend

This parameter tells the system what action to take when the system has detected that it is low on disk space. Valid values are ignore, syslog, rotate, email, exec, suspend, single, and halt.

    audit_disk_full_action: SUSPEND

This parameter tells the system what action to take when the system has detected that the partition to which log files are written has become full. Valid values are ignore, syslog, rotate, exec, suspend, single, and halt.

    audit_disk_error_action: SUSPEND

This parameter tells the system what action to take whenever there is an error detected when writing audit events to disk or rotating logs. Valid values are ignore, syslog, exec, suspend, single, and halt.

    #audit_tcp_listen_port: 60

This is a numeric value in the range 1..65535 which, if specified, causes auditd to listen on the corresponding TCP port for audit records from remote systems. The audit daemon may be linked with tcp_wrappers. You may want to control access with an entry in the hosts.allow and deny files. If this is deployed on a systemd based OS, then you may need to adjust the 'After' directive.

    #audit_tcp_listen_queue: 5

This is a numeric value which indicates how many pending (requested but unaccepted) connections are allowed.  The default is 5.  Setting this too small may cause connections to be rejected if too many hosts start up at exactly the same time, such as after a power failure. This setting is only used for aggregating servers. Clients logging to a remote server should keep this commented out.

    #audit_tcp_max_per_addr: 1

This is a numeric value which indicates how many concurrent connections from one IP address is allowed. The default is 1 and the maximum is 1024. Setting this too large may allow for a Denial of Service attack on the logging server.

    audit_use_libwrap: true

This setting determines whether or not to use tcp_wrappers to discern connection attempts that are from allowed machines.

    #audit_tcp_client_ports: 1024-65535

This parameter may be a single numeric value or two values separated by a dash (no spaces allowed).  It indicates which client ports are allowed for incoming connections. If not specified, any port is allowed. 

    audit_tcp_client_max_idle: 0

This parameter indicates the number of seconds that a client may be idle (i.e. no data from them at all) before auditd complains. This is used to close inactive connections if the client machine has a problem where it cannot shutdown the connection cleanly. Note that this is a global setting, and must be higher than any individual client heartbeat_timeout setting, preferably by a factor of two.  The default is zero, which disables this check.

    audit_transport: TCP

If set to TCP, only clear text tcp connections will be used. If set to KRB5, then Kerberos 5 will be used for authentication and encryption.

    audit_krb5_principal: auditd

This is the principal for this server.  The default is "auditd".  Given this default, the server will look for a key named like auditd/hostname@EXAMPLE.COM stored in /etc/audit/audit.key to authenticate itself, where hostname is the canonical name for the server's host, as returned by a DNS lookup of its IP address.

    #audit_krb5_key_file: /etc/audit/audit.key

Location of the key for this client's principal.  Note that the key file must be owned by root and mode 0400.

    audit_distribute_network: false

If set to true, network originating events will be distributed to the audit dispatcher for processing.

    audit_q_depth: 400

This is a numeric value that tells how big to make the internal queue of the audit event dispatcher. A bigger queue lets it handle a flood of events better, but could hold events that are not processed when the daemon is terminated. If you get messages in syslog about events getting dropped, increase this value.

    audit_overflow_action: SYSLOG

This option determines how the daemon should react to overflowing its internal queue. When this happens, it means that more events are being received than it can pass along to child processes. This error means that it is going to lose the current event that it's trying to dispatch. This option has the following choices: ignore, syslog, suspend, single, and halt.

    audit_max_restarts: 10

This is a non-negative number that tells the audit event dispatcher how many times it can try to restart a crashed plugin.

    audit_plugin_dir: /etc/audit/plugins.d

This is the location that auditd will use to search for its plugin configuration files.

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

    audit_rules_files:
      - 01-init.rules
      - 10-self-audit.rules
      - 20-filters.rules
      - 30-kernel.rules
      - 40-identity.rules
      - 40-login.rules
      - 40-mount.rules
      - 40-stunnel.rules
      - 40-swap.rules
      - 40-time.rules
      - 50-cron.rules
      - 50-dac.rules
      - 50-hostname.rules
      - 50-ip-connections.rules
      - 50-network.rules
      - 50-pkg-manager.rules
      - 50-remote-shell.rules
      - 50-sudoers.rules
      - 50-system-libs.rules
      - 50-system-startup.rules
      - 55-privileged.rules
      - 60-mail.rules
      - 60-pam.rules
      - 60-sshd.rules
      - 60-systemd.rules
      - 70-access.rules
      - 70-mac-policy.rules
      - 70-power-state.rules
      - 70-sessions.rules
      - 70-shell-profiles.rules
      - 80-data-compression.rules
      - 80-privilege-abuse.rules
      - 80-reconnaissance.rules
      - 80-socket-creation.rules
      - 80-suspicious.rules
      - 80-suspicious-shells.rules
      - 85-network.rules
      - 85-virtualization.rules
      - 90-32bit-api-exploitation.rules

Specify the rules' files to be copied. The files listed above are available by default, but user can create their own files as needed. The role provides the rules separated in several small files (instead of a big one), to promote re-usability: you can select wich rules to implement, and you can create your own custom rules.

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
