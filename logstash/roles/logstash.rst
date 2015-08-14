.. _logstash_role:

logstash
=========

.. contents:: Topics

Setup logstash with a tarball and running it as a service.

Requirements
------------

Ansible-core-modules: ``template`` , ``file`` , ``stat`` , ``unarchive`` , ``lineinfile``

Role Variables
--------------

Default vars::

  logstash_srvname: "logstash"
  base_home: "{{ansible_env.HOME}}/lek"
  logstash_tarball: "logstash-1.5.0.tar.gz"
  logstash_home: "{{base_home}}/logstash"
  env_profile: "{{ansible_env.HOME}}/.profile"
  logstash_whoami: "shipper"
  read_from_redis_addr: "127.0.0.1"
  read_from_redis_port: 6379
  write_to_redis_addr: "127.0.0.1"
  write_to_redis_port: 6379
  redis_key: "logstash-*"
  els_addr: "127.0.0.1"
  operation: "install"

OS-specified vars::

  RedHat-based
    default_syslog: ["/var/log/messages"]
    default_authlog: ["/var/log/secure"]

  Debian-based
    default_syslog: [ "/var/log/syslog", "/var/log/kern.log" ]
    default_authlog: [ "/var/log/authlog" ]

Dependencies
------------

None

Example Playbook
----------------

::

  - hosts: servers
    gather_facts: true
    roles:
       - { role: logstash, logstash_whoami: "shipper", \
           write_to_redis_addr: "127.0.0.1", operation: "install" }
