.. _sun-jdk:

sun-jdk
=========

.. contents:: Topics

Install sun-jdk-8u45

Requirements
------------

Ansible-core-module: ``file`` , ``unarchive`` , ``stat`` , ``lineinfile`` .

Role Variables
--------------

Default vars::

  base_home: "{{ansible_env.HOME}}/lek"
  java_home: "{{base_home}}/jdk1.8.0_45"
  jdk_tarball: "jdk-8u45-linux-x64.tar.gz"

Dependencies
------------

None

Example Playbook
----------------

::

  - hosts: servers
    gather_facts: true
    roles:
      - { role: sun-jdk }
