.. _elasticsearch_role:

elasticsearch
================

.. contents:: Topics

Setup a single node elasticsearch service on remote server.

Requirements
------------

Ansible-core-modules: ``template`` , ``file`` , ``unarchive`` , ``stat`` , ``lineinfile`` .

Role Variables
--------------

Default vars::

  base_home: "{{ansible_env.HOME}}/lek"
  es_home: "{{base_home}}/elasticsearch"
  es_tarball: "elasticsearch-1.6.0.tar.gz"
  es_config: "elasticsearch.yml"
  es_logging_config: "logging.yml"
  es_srvname: "elasticsearch"
  env_profile: "{{ansible_env.HOME}}/.profile"
  operation: "install"

Dependencies
------------

JDK

.. note::

  JDK 1.8 or later required. 
  Make sure executable java binary installed in /bin:/usr/bin:/sbin:/usr/sbin, or make a symbolic link in /bin:/usr/bin:/sbin:/usr/sbin .

Example Playbook
----------------

::

  - hosts: servers
    gather_facts: true
    roles:
       - { role: elasticsearch, operation: "install" }
