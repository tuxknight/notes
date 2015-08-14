.. _lek_playbooks:

Playbooks
==============

.. contents:: Topics


Linear Execution
-------------------

#. Common settings

#. Setup services

#. Setup applications

Privilege Problems
--------------------

Controlling machine have the permission to connect remote nodes as normal users. [ elc, ats, susu ]

But some tasks require root privileges. If you connect to nodes as root, applications you installed will running with root privileges. It is better that using sudo & sudo_user in each task which needs root privileges.

::

  tasks:
    - name: root privilege required
      apt: pkg="redis-server" state=present
      sudo: yes
      sudo_user: root

*ansible-playbook -i hosts site.yml --ask-sudo-pass -u elc* This command will prompt for a password of elc on every hosts. When you connect to remote nodes with various users and passwords, using group_vars and hosts_vars is a convenient approach to manage each node's username and passwords.

::

  ---
  ansible_ssh_user: susu
  ansible_sudo_pass: ********

*ansible-vault* allows keeping sensitive data such as passwords or keys in encrypted files.

::

  ansible-vault create host_vars/10.32.131.107.yml
  
  ansible-vault edit host_vars/10.32.131.107.yml --ask-vault-pass
  
  ansible-vault view host_vars/10.32.131.107.yml --vault-password-file /some/safe/place/pass.txt
  
  ansible-vault rekey host_vars/10.32.131.107.yml

Inventory
--------------

Inventory file::

  [lek:children]
  logstash
  els
  
  [logstash:children]
  shipper
  indexer
  
  [els]
  10.13.181.85
  
  [shipper]
  10.32.105.214
  10.32.131.107
  
  [indexer]
  10.32.105.213
  
  [redis]
  10.32.105.213
  

Playbook site.xml ::

  ---
  - name: common settings
    hosts: lek
    gather_facts: true
    roles: 
      - {role: common-env}
      - {role: sun-jdk}
  
  - name: setup redis
    hosts: redis
    gather_facts: true
    roles: 
      - {role: redis}
        
  - name: setup kibana and elasticsearch
    hosts: els
    gather_facts: true
    roles: 
      - {role: elasticsearch}
      - {role: kibana}
  
  # when redis and elasticsearch are ready
  - name: setup logstash
    hosts: logstash
    gather_facts: true
    roles: 
      - {role: logstash}

