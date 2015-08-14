.. _redis_roles:

redis
=========

.. contents:: Topics

Setup redis on remote servers with redis-3.0.2 source code.

Requirements
------------

Ansible-core-modules: ``unarchive`` , ``stat`` , ``shell`` , ``template`` , ``lineinfile`` , ``file`` .

Role Variables
--------------

Default vars::

  base_home: "{{ansible_env.HOME}}/lek"
  redis_home: "{{base_home}}/redis"
  redis_tarball: "redis-3.0.2.tar.gz"
  redis_config: "redis_{{redis_port}}.conf"
  redis_init: "redis_{{redis_port}}_init"
  redis_port: 6379
  redis_srvname: "redis_{{redis_port}}"

Dependencies
------------

None

Example Playbook
----------------

::

 - hosts: servers
   gather_facts: true
   roles:
      - { role: redis, redis_port: 6379 }
      - { role: redis, redis_port: 6380 }
