Neutron-logging
=====================

.. contents:: Topics

Send logging information to syslog

Requirements
------------

Ansible-core-module: ``lineinfile``

Role Variables
--------------

Default vars:
  neutron_conf: "/etc/neutron/neutron.conf"
  neutron_facility:
    LOG_LOCAL2: local2
  port: "10514"
  host: "127.0.0.1"
  log_server: "{{host}}:{{port}}"
  protocal: "tcp"
  lines:
    - {regx: "^verbose=", line: "verbose=False" }
    - {regx: "^debug=", line: "debug=False" }
    - {regx: "^use_syslog=", line: "use_syslog=True" }
    - {regx: "^syslog_log_facility=", line: "syslog_log_facility=LOG_LOCAL2" }

Dependencies
------------

None

Example Playbook
----------------

::

    - hosts: servers
      roles:
         - { role: neutron-logging, host: "10.32.10.153" }

License
-------

BSD

