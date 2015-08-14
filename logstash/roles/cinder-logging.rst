Cinder-logging
==================

.. contents:: Topics

Sending logging information to syslog

Requirements
------------

Ansible-core-modules: ``lineinfile``

Role Variables
--------------

Default vars::

  cinder_conf: "/etc/cinder/cinder.conf"
  cinder_facility:
    LOG_LOCAL1: local1
  port: "10514"
  host: "127.0.0.1"
  log_server: "{{host}}:{{port}}"
  protocal: "tcp"
  lines:
    - {regx: "^verbose=", line: "verbose=False" }
    - {regx: "^debug=", line: "debug=False" }
    - {regx: "^use_syslog=", line: "use_syslog=True" }
    - {regx: "^syslog_log_facility=", line: "syslog_log_facility=LOG_LOCAL1" }

Dependencies
------------

None

Example Playbook
----------------

::

    - hosts: servers
      roles:
         - { role: cinder-logging, port: "11514" }

License
-------

BSD
