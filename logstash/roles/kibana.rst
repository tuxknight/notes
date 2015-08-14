.. _kibana_role:

kibana
=========

.. contents:: Topics

Setup Kibana on remote server.

Requirements
------------

Ansible-core-module: ``file`` , ``unarchive`` , ``lineinfile`` , ``template`` , ``stat`` .

Role Variables
--------------

Default vars::

  base_home: "{{ansible_env.HOME}}/lek"
  kibana_home: "{{base_home}}/kibana"
  kibana_tarball: "kibana-4.1.0-linux-x64.tar.gz"
  env_profile: "{{ansible_env.HOME}}/.profile"
  kibana_config: "kibana.yml"
  elasticsearch_url: "http://127.0.0.1:9200"
  kibana_port: 5601
  operation: "install"

Dependencies
------------

None

Example Playbook
----------------

::

  - hosts: servers
    gather_facts: true
    roles:
       - { role: kibana, elasticsearch_url:"http://10.13.181.85:9200", \
           operation: "install" }
