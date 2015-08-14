.. _ansible_cfg:

============================
Ansible Configuration File
============================

Configuration file will processed in the following order::

  * ANSIBLE_CONFIG (an environment varialbe)
  * ansible.cfg (in the current directory)
  * .ansible.cfg (in the home directory)
  * /etc/ansible/ansible.cfg

Ansible will process the above list and use the first file found. Setting s in files are not merged.

If you installed Ansible from pip or from source, you may want to create this file in order to override default settings in Ansible.

You may wish to consult the `ansible.cfg`_ in source control for all of the possible lastest values.

.. _`ansible.cfg`: https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg


> ansible_managed
===================

  Ansible-managed is a string that can be inserted into files written by Ansible’s config templating system, if you use a string like::

    {{ ansible_managed }}

  The default configuration shows who modified a file and when::

    ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host}

  This is useful to tell users that a file has been placed by Ansible and manual changes are likely to be overwritten.

  Note that if using this feature, and there is a date in the string, the template will be reported changed each time as the date is updated.


> ask_pass
================

  This controls whether an Ansible playbook should prompt for a password by default. The default behavior is no.


> ask_sudo_pass
==================

  Similar to ask_pass, this controls whether an Ansible playbook should prompt for a sudo password by default when sudoing. The default behavior is also no::

    ask_sudo_pass=True

  Users on platforms where sudo passwords are enabled should consider changing this setting.


> bin_ansible_callbacks
===========================

  Controls whether callback plugins are loaded when running /usr/bin/ansible. This may be used to log activity from the command line, send notifications, and so on. Callback plugins are always loaded for /usr/bin/ansible-playbook if present and cannot be disabled.


> command_warnings
========================

  By default since Ansible 1.8, Ansible will warn when usage of the shell and command module appear to be simplified by using a default Ansible module instead. This can include reminders to use the ‘git’ module instead of shell commands to execute ‘git’. Using modules when possible over arbitrary shell commands can lead to more reliable and consistent playbook runs, and also easier to maintain playbooks::

     command_warnings=False

  These warnings can be silenced by adjusting the following setting or adding warn=yes or warn=no to the end of the command line parameter string, like so::

    - name: usage of git that could be replaced with the git module
      shell: git update foo warn=yes

> deprecation_warnings
==========================

  Allows disabling of deprecating warnings in ansible-playbook output::

    deprecation_warnings = True

> display_skipped_hosts
============================

  If set to False, ansible will not display any status for a task that is skipped. The default behavior is to display skipped tasks::

    display_skipped_hosts=True

  Note that Ansible will always show the task header for any task, regardless of whether or not the task is skipped.

> error_on_undefined_vars
=============================

  On by default since Ansible 1.3, this causes ansible to fail steps that reference variable names that are likely typoed::

    error_on_undefined_vars=True

  If set to False, any ‘{{ template_expression }}’ that contains undefined variables will be rendered in a template or ansible action line exactly as written.

> force_color
================

  This option forces color mode even when running without a TTY::

    force_color=1

> force_handlers
================

  This option causes notified handlers to run on a host even if a failure occurs on that host::

    force_handlers=True

  The default is False, meaning that handlers will not run if a failure has occurred on a host.

> forks
================

  This is the default number of parallel processes to spawn when communicating with remote hosts. Many users may set this to 50, some set it to 500 or more. The default is very very conservative::

    forks=5

> timeout
================

  This is the default SSH timeout to use on connection attempts::

    timeout=10

> gathering
================

  The ‘gathering’ setting controls the default policy of facts gathering (variables discovered about remote systems).

  The value ‘implicit’ is the default, meaning facts will be gathered per play unless ‘gather_facts: False’ is set in the play. The value ‘explicit’ is the inverse, facts will not be gathered unless directly requested in the play.

  The value ‘smart’ means each new host that has no facts discovered will be scanned, but if the same host is addressed in multiple plays it will not be contacted again in the playbook run. This option can be useful for those wishing to save fact gathering time.

> host_key_checking
=====================

  If a host is reinstalled and has a different key in ‘known_hosts’, this will result in an error message until corrected. If a host is not initially in ‘known_hosts’ this will result in prompting for confirmation of the key, which results in an interactive experience if using Ansible, from say, cron. You might not want this.

> poll_interval
================

  For asynchronous tasks in ansible, this is how often to check back on the status of those tasks when an explicit poll interval is not supplited::

    poll_interval=15

> inventory
================

  This is the default location of the inventory file, script, or directory that Ansible will use to determine what hosts it has available to talk to::

    inventory=/etc/ansible/hosts

> jinja2_extensions
=======================

  This is a developer-specific feature that allows enabling additional Jinja2 extensions::

    jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n


> library
================

  This is the default location Ansible looks to find modules::

    library=/usr/share/ansible:~/ansible

  It also will look for modules in the './library' directory alongside a playboogk.

> log_path
================

  If present and configured in ansible.cfg, Ansible will log information about executions at the designated location. Be sure the user running Ansible has permissions on the logfile::

    log_path=/var/log/ansible.log

  This behavior is not on by default. Note that ansible will, without this setting, record module arguments called to the syslog of managed machines. Password arguments are excluded.

> module_lang
================

  This is to set the default language to communicate between the module and the system. By default, the value is ‘C’.

> module_name
================

  This is the default module name (-m) value for /usr/bin/ansible. The default is the 'command' module. This module doesnot support shell variables, pipes, or quotes, so you might wish to change it to 'shell'::

    module_name=command

> nocolor
================

> nocows
================

> pattern
================

  This is the default group of hosts to talk to in a playbook if no “hosts:” stanza is supplied. The default is to talk to all hosts. You may wish to change this to protect yourself from surprises::

    pattern=*

  This is only used in ``ansible-playbook`` , and ``ansible`` always requires a host pattern.

> private_key_file
=====================

  If you are using a pem file to authenticate with machines rather than SSH agent or passwords, you can set the default value here to avoid re-specifying ``================private-key`` with every invocation::

    private_key_file=/path/to/file.pem

> remote_port
================

  This sets the default SSH port on all of your systems, for systems that didn’t specify an alternative value in inventory. The default is the standard 22::

    remote_port=22

> remote_tmp
================

  Ansible works by transferring modules to your remote machines, running them, and then cleaning up after itself. In some cases, you may not wish to use the default location and would like to change the path. You can do so by altering this setting::

    remote_tmp=$HOME/.ansible/tmp

> remote_user
================

  This is the default username ansible will connect as for /usr/bin/ansible-playbook. Note that /usr/bin/ansible will always default to the current user if this is not defined::

    remote_user=root

> roles_path
================

  The roles path indicate additional directories beyond the ‘roles/’ subdirectory of a playbook project to search to find Ansible roles. For instance, if there was a source control repository of common roles and a different repository of playbooks, you might choose to establish a convention to checkout roles in /opt/mysite/roles like so::

    roles_path=/opt/mysite/roles:/etc/ansible/roles

> sudo_exe
================

  If using an alternative sudo implementation on remote machines, the path to sudo can be replaced here provided the sudo implementation is matching CLI flags with the standard sudo:

    sudo_exe=sudo

> sudo_flags
================

  Additional flags to pass to sudo when engaging sudo support. The default is ‘-H’ which preserves the $HOME environment variable of the original user. In some situations you may wish to add or remove flags, but in general most users will not need to change this setting::

    sudo_flags=-H

> sudo_user
================

  This is the default user to sudo to if ``================sudo-user`` is not specified or 'sudo_user' is not specified in an Ansible playbook::

    sudo_user=root

> become
================

  The equivalent of adding sudo: or su: to a play or task, set to true/yes to activate privilege escalation. The default behavior is no::

    become=True

> become_method
=================

  Set the privilege escalation method. The default is ``sudo`` , other options are ``su`` , ``pbrun`` , ``pfexec`` ::

    become_method=su

> become_user
================

  The equivalent to ansible_sudo_user or ansible_su_user, allows to set the user you become through privilege escalation. The default is ‘root’::

    become_user=root

> become_ask_pass
===================

  Ask for privilege escalation password, the default is False::

    become_ask_pass=True

> record_host_keys
====================

  The default setting of yes will record newly discovered and approved (if host key checking is enabled) hosts in the user’s hostfile. This setting may be inefficient for large numbers of hosts, and in those situations, using the ssh transport is definitely recommended instead. Setting it to False will improve performance and is recommended when host key checking is disabled::

    record_host_keys=True

> ssh_args
================

  If set, this will pass a specific set of options to Ansible rather than Ansible’s usual defaults::

    ssh_args=-o ControlMaster=auto -o ControlPersist=60s

  In particular, users may wish to raise the ControlPersist time to encourage performance. A value of 30 minutes may be appropriate.


> control_path
================

  This is the location to save ControlPath sockets. This defaults to::

    control_path=%(directory)s/ansible-ssh-%%h-%%p-%%r

> scp_if_ssh
================

  Occasionally users may be managing a remote system that doesn’t have SFTP enabled. If set to True, we can cause scp to be used to transfer remote files instead::

    scp_if_ssh=False

> pipelining
================

  Enabling pipelining reduces the number of SSH operations required to execute a module on the remote server, by executing many ansible modules without actual file transfer. This can result in a very significant performance improvement when enabled, however when using “sudo:” operations you must first disable ‘requiretty’ in /etc/sudoers on all managed hosts.

  By default, this option is disabled to preserve compatibility with sudoers configurations that have requiretty, but is highly recommended if yo can enable it.

> accelerate_port
=====================

  This is the port to use for accelerated mode::

    accelerate_port=5099

> accelerate_timeout
===========================

  This setting controls the timeout for receiving data from a client. If no data is received during this time, the socket connection will be closed. A keepalive packet is sent back to the controller every 15 seconds, so this timeout should not be set lower than 15 (by default, the timeout is 30 seconds)::

    accelerate_timeout=30

> accelerate_connect_timeout
==============================

  This setting controls the timeout for the socket connect call, and should be kept relatively low. The connection to the accelerate_port will be attempted 3 times before Ansible will fall back to ssh or paramiko (depending on your default connection setting) to try and start the accelerate daemon remotely. The default setting is 1.0 seconds::

    accelerate_connect_timeout=1.0

> accelerate_multi_key
==========================

  If enabled, this setting allows multiple private keys to be uploaded to the daemon. Any clients connecting to the daemon must also enable this option::

    accelerate_multi_key=yes
