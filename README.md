Ansible Role: openssh
=========

An Ansible Role that install and configure openssh on RHEL/CentOS, Fedora and Debian/Ubuntu. The values provided are based on CIS and other sites recommendations.

Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    openssh_port: 22

SSH service port number. This role is able to change port number but depending on the situation it may fail (firewall problems, selinux, etc).

    openssh_log_level: INFO

Set log level

    openssh_x11forwarding: no

Enable (true/yes) or disable (false/no) X11 forwarding

    openssh_max_auth_tries: 4

Specifies the maximum number of authentication attempts permitted per connection. Once the number of failures reaches half this value, additional failures are logged.

    openssh_client_alive_interval: 300

Sets a timeout interval in seconds after which if no data has been received from the client, sshd will send a message through the encrypted channel to request a response from the client.

    openssh_client_alive_count_max: 0

Sets the number of client alive messages (see below) which may be sent without sshd(8) receiving any messages back from the client. If this threshold is reached while client alive messages are being sent, sshd will disconnect the client, terminating the session. It is important to note that the use of client alive messages is very different from TCPKeepAlive (below). The client alive messages are sent through the encrypted channel and therefore will not be spoofable. The TCP keepalive option enabled by TCPKeepAlive is spoofable. The client alive mechanism is valuable when the client or server depend on knowing when a connection has become inactive. 

    openssh_login_grace_time: 60

The server disconnects after this time if the user has not successfully logged in. If the value is 0, there is no time limit.

    openssh_banner: /etc/issue.net

The contents of the specified file are sent to the remote user before authentication is allowed. If the argument is ''none'' then no banner is displayed. This option is only available for protocol version 2.

    openssh_allow_tcp_forwarding: no

Specifies whether TCP forwarding is permitted. Note that disabling TCP forwarding does not improve security unless users are also denied shell access, as they can always install their own forwarders. 

    openssh_max_sessions: 4

Specifies the maximum number of open sessions permitted per network connection.

    openssh_ignore_rhosts: yes

Specifies that .rhosts and .shosts files will not be used in RhostsRSAAuthentication or HostbasedAuthentication. 

    openssh_host_based_authentication: no

Specifies whether rhosts or /etc/hosts.equiv authentication together with successful public key client host authentication is allowed (host-based authentication). This option is similar to RhostsRSAAuthentication and applies to protocol version 2 only.

    openssh_permit_root_login: no

Specifies whether root can log in using ssh. The argument must be 'yes', 'without-password', 'forced-commands-only', or 'no'. If this option is set to ''without-password'', password authentication is disabled for root. If this option is set to 'forced-commands-only', root login with public key authentication will be allowed, but only if the command option has been specified (which may be useful for taking remote backups even if root login is normally not allowed). All other authentication methods are disabled for root. If this option is set to 'no', root is not allowed to log in. 

    openssh_permit_empty_passwords: no

When password authentication is allowed, it specifies whether the server allows login to accounts with empty password strings.

    openssh_permit_user_environment: no

Specifies whether ~/.ssh/environment and environment= options in ~/.ssh/authorized_keys are processed by sshd(8). Enabling environment processing may enable users to bypass access restrictions in some configurations using mechanisms such as LD_PRELOAD. 

    openssh_use_pam: yes

Enables the Pluggable Authentication Module interface. If set to 'yes' this will enable PAM authentication using ChallengeResponseAuthentication and PasswordAuthentication in addition to PAM account and session module processing for all authentication types. 
Because PAM challenge-response authentication usually serves an equivalent role to password authentication, you should disable either PasswordAuthentication or ChallengeResponseAuthentication. 

If UsePAM is enabled, you will not be able to run sshd as a non-root user.

    openssh_max_startups: '10:30:60'

Specifies the maximum number of concurrent unauthenticated connections to the SSH daemon. Additional connections will be dropped until authentication succeeds or the LoginGraceTime expires for a connection.
Alternatively, random early drop can be enabled by specifying the three colon separated values 'start:rate:full' (e.g. "10:30:60"). sshd will refuse connection attempts with a probability of 'rate/100' (30%) if there are currently 'start' (10) unauthenticated connections. The probability increases linearly and all connection attempts are refused if the number of unauthenticated connections reaches 'full' (60).

    openssh_use_dns: yes

 Specifies whether sshd(8) should look up the remote host name and check that the resolved host name for the remote IP address maps back to the very same IP address.

    openssh_password_authentication: yes

Specifies whether password authentication is allowed.

    openssh_permit_tunnel: no

Specifies whether tun device forwarding is allowed. The argument must be 'yes', 'point-to-point' (layer 3), 'ethernet' (layer 2), or 'no'. Specifying 'yes' permits both 'point-to-point' and 'ethernet'.

    #openssh_allow_users: ['myuser']

This keyword can be followed by a list of user name patterns, separated by spaces. If specified, login is allowed only for user names that match one of the patterns. Only user names are valid; a numerical user ID is not recognized. By default, login is allowed for all users. If the pattern takes the form USER@HOST then USER and HOST are separately checked, restricting logins to particular users from particular hosts. The allow/deny directives are processed in the following order: DenyUsers, AllowUsers, DenyGroups, and finally AllowGroups. 

    #openssh_allow_groups: []

This keyword can be followed by a list of group name patterns, separated by spaces. If specified, login is allowed only for users whose primary group or supplementary group list matches one of the patterns. Only group names are valid; a numerical group ID is not recognized. By default, login is allowed for all groups. The allow/deny directives are processed in the following order: DenyUsers, AllowUsers, DenyGroups, and finally AllowGroups. 

    #openssh_deny_users: []

 This keyword can be followed by a list of user name patterns, separated by spaces. Login is disallowed for user names that match one of the patterns. Only user names are valid; a numerical user ID is not recognized. By default, login is allowed for all users. If the pattern takes the form USER@HOST then USER and HOST are separately checked, restricting logins to particular users from particular hosts. The allow/deny directives are processed in the following order: DenyUsers, AllowUsers, DenyGroups, and finally AllowGroups. 

    #openssh_deny_groups: []

This keyword can be followed by a list of group name patterns, separated by spaces. Login is disallowed for users whose primary group or supplementary group list matches one of the patterns. Only group names are valid; a numerical group ID is not recognized. By default, login is allowed for all groups. The allow/deny directives are processed in the following order: DenyUsers, AllowUsers, DenyGroups, and finally AllowGroups. 

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    host_keys_group:

Group that owns host keys. It changes based on operating system.

    host_keys_mode:

Permissions to be set on host keys. It changes based on operating system.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      vars:
        openssh_port: 10999
        openssh_allow_users['user1', 'user2']
      roles:
         - { role: guidugli.openssh }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
