.. _playbooks_intro:

===========
Playbooks
===========

.. contents:: Topics

`Playbooks`_ examples

.. _Playbooks: https://github.com/ansible/ansible-examples

每个playbook 至少包含一个 play列表，play的作用就是把一组主机和一组定义好的roles关联起来。

最基本的task就是直接调用ansible的模块::

  ---
  - hosts: web
    name: install httpd
    gather_facts: true

    tasks: 
      - name: install httpd
        yum: name={{ item }} state=installed
        with_items:
          - httpd
          - httpd-devel
        when: ansible_os_family == "Redhat"
        notify:
          - restart apache2

      - name: install httpd
        apt: name={{ item }} state=installed
        with_items:
          - apache2
          - apache2-dev
        when: ansible_os_family == "Debian"
        notify:
          - restart apache2

    handlers:
      - name: restart httpd
        service: 
          name=httpd 
          state=restarted
        when: ansible_os_family == "Debian"

      - name: restart apache2
        service: name=apache2 state=restarted
        when: ansible_os_family == "Debian"


Hosts and Users
=================

::

  ---
  - hosts: webservers
    remote_user: root

The *hosts* line is a list of one or more groups or host patterns,separated by colons.

The *remote_user* is just the name of the user account.

Remote users can also be defined per task::

  ---
  - hosts: webserver
    remote_user: root
    tasks:
      - name: test connection
        ping: 
        remote_user: localuser

Support for running things as another user is also available,you could become a user as root or other different user::

  ---
  - hosts: webserver
    remote_user: root
    tasks:
      - name: restart httpd
        service: name=httpd state=restarted
        remote_user: localuser
        become: yes
        become_method: sudo

      - name: restart mysql
        service: name=mysqld state=restarted
        remote_user: localuser
        become: yes
        become_user: mysql
        become_method: su


Tasks list
============

Within a play, all hosts are going to get the same task directives. It is the purpose of a play to map a selection of hosts to tasks.

Hosts with failed tasks are taken out of the rotation for the entire playbook. If things fail, simply correct the playbook file and rerun.

The goal of each task is to execute a module with very specific arguments.

Modules are 'idempotent' (幂等) , meaning if you run them again, they will make only the changes they must in order to bring the system to the desired state. This makes it very safe to rerun the same playbook multiple times.

Every task should have a *name* , which is included in the output from running the playbook. If the name is not provided though, the string fed to 'action' will be used for output.

::

  tasks: 
    - name: make sure apache is running
      service: name=httpd state=running
    - name: disable selinux
      command: /sbin/setenforce 0

*command* and *shell* module care about return codes, so if you have a command whose successful exit code is not zero, you may wish to do this::

  tasks:
    - name: run this command and ignore the result
      shell: /usr/bin/somecommand || /bin/true

Or this::

  tasks:
    - name: run this command and ignore the result
      shell: /usr/bin/somecommand
      ignore_errors: True

Variables can be used in action lines::

  tasks:
    - name: create a virtual host file for {{ vhost }}
      template: src=somefile.j2 dest=/etc/httpd/conf.d/{{ vhost }}

Handlers: Running Operations On change
=========================================

Playbooks recognize changes and have a basic event system that can be used to respond to change.

These *notify* actions are triggered at the end of each block of tasks in a playbook, and will only be triggered once even if notified by multiple different tasks::

  - name: template configuration file
    template: src=template.j2 dest=/etc/foo.conf
    notify:
      - restart memcached
      - restart apache

The things listed in the *notify* section of a task are called handler.

Handlers are lists of tasks, not really any different from regular tasks, that are referenced by name::

  handlers:
    - name: restart memcached
      service: name=memcached state=restarted
    - name: restart apache
      service: name=apache state=restarted

Handlers are best used to restart services and trigger reboots.

Handlers are automatically processed between 'pre_tasks','roles','tasks','post_tasks' sections. if you ever want to flush all handler commands immediately though::

  tasks:
    - shell: some tasks go here
    - meta: flush_handlers
    - shell: some other tasks

Executing A Playbook
======================

``ansible-playbook playbook.yml -f 10``

Ansible-Pull
==============

ansible-pull is a small script that will checkout a repo of configuration instructions from git, and the run ansible-playbook against that content.

Tips and Tricks
================

If you ever wanted to see detailed output from successful modules as well as  unsuccessful ones, use ``--verbose`` flag.

To see what hosts would be affected by a playbook before you run it, you can do this::

  ansible-playbook playbook.yml --list-hosts
