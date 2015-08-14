Nova-logging
===============

.. contents:: Topics

Send logging information to syslog

Requirements
------------

Ansible-core-modules: ``lineinfile``

Role Variables
--------------

Default vars::

  nova_conf: "/etc/nova/nova.conf"
  nova_facility: 
    LOG_LOCAL0: local0
  host: "127.0.0.1"
  port: "10514"
  log_server: "{{host}}:{{port}}"
  protocol: "tcp"
  lines:
    - {regx: "^verbose=", line: "verbose=False" }
    - {regx: "^debug=", line: "debug=False" }
    - {regx: "^use_syslog=", line: "use_syslog=True" }
    - {regx: "^syslog_log_facility=", line: "syslog_log_facility=LOG_LOCAL0" }


Dependencies
------------

None

Example Playbook
----------------

::

    - hosts: servers
      roles:
         - { role: nova-logging, log_server: "10.32.105.153:10514"}

License
-------

BSD
