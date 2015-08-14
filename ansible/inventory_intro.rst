.. _inventory_intro:

==============
Inventory
==============

.. contents:: Topics

Multiple inventory files can be used at the same time and also pull inventory from ``dynamic`` or ``cloud sources`` as described in :ref:`dynamic_inventory` .


Hosts and Groups
==================

``/etc/ansible/hosts`` is an INI-like format hostfile.To make things explicit, you could describe hosts like this::

  [webserver]
  ws01.bj.example.com
  ws02.bj.example.com:3322
  [dbserver]
  ws01.bj.example.com

  [ops]
  jumper ansible_ssh_port=6222 ansible_ssh_host=192.168.1.80

Patterns like ``[1-5] [a-z]`` can be included to define large range of hosts.

::

  [lbserver]
  lvs[a-c][00:21].example.com

Host Variables
================

It is easy to assign variables to hosts that will be used later in playbooks::

  [vast]
  server01 app_port=3876 Maxrequests=300
  server03 app_port=3876 Maxrequests=500

Group Variables
=================

However, variables can also be applied to an entire group at once::

  [vast]
  server01
  server02
  server03

  [vast:vars]
  proxy=proxy.local.com
  nextJump=next.local.com

Groups of Groups, and Group Variables
=========================================

It is possible to make groups of groups and assign variables to groups. These variables can be used by ``ansible-playbook`` , but not ``ansible`` .

::

  [region01]
  server01
  server02

  [region02]
  serverA
  serverB

  [region03]
  serverX
  serverY

  [large:children]
  region01
  region02

  [large:vars]
  hostDistance=2
  allowFailure=False

Splitting Out Host and Group Specific Data
============================================

The preferred practice in Ansible is actually not to store variables in the main inventory file. Host and group variables can be stored in individual files relative to the inventory file.

Assuming the inventory file path is::

  /etc/ansible/hosts

You can assign varialbes in ``/etc/ansible/group_vars/region02`` ``/etc/ansible/host_vars/server01``

List of Behavioral Inventory Parameters
=========================================

::


  ansible_ssh_host
    The name of the host to connect to, if different from the alias you wish to give to it.
  ansible_ssh_port
    The ssh port number, if not 22
  ansible_ssh_user
    The default ssh user name to use.
  ansible_ssh_pass
    The ssh password to use (this is insecure, we strongly recommend using --ask-pass or SSH keys)
  ansible_sudo
    The boolean to decide if sudo should be used for this host. Defaults to false.
  ansible_sudo_pass
    The sudo password to use (this is insecure, we strongly recommend using --ask-sudo-pass)
  ansible_sudo_exe (new in version 1.8)
    The sudo command path.
  ansible_connection
    Connection type of the host. Candidates are local, ssh or paramiko.  The default is paramiko before Ansible 1.2, and 'smart' afterwards which detects whether usage of 'ssh' would be feasible based on whether ControlPersist is supported.
  ansible_ssh_private_key_file
    Private key file used by ssh.  Useful if using multiple keys and you don't want to use SSH agent.
  ansible_shell_type
    The shell type of the target system. Commands are formatted using 'sh'-style syntax by default. Setting this to 'csh' or 'fish' will cause commands executed on target systems to follow those shell's syntax instead.
  ansible_python_interpreter
    The target host python path. This is useful for systems with more
    than one Python or not located at "/usr/bin/python" such as \*BSD, or where /usr/bin/python
    is not a 2.X series Python.  We do not use the "/usr/bin/env" mechanism as that requires the remote user's
    path to be set right and also assumes the "python" executable is named python, where the executable might
    be named something like "python26".
  ansible\_\*\_interpreter
    Works for anything such as ruby or perl and works just like ansible_python_interpreter.
    This replaces shebang of modules which will run on that host.
