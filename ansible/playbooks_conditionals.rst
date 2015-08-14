.. _playbooks_conditionals:

==============
Conditionals
==============

.. contents:: Topics

The When Statement
====================

With the **when** clause, which contains a Jinja2 expression, It is easy to do something particular on a particular host::

  tasks:
    - name: "shutdown CentOS 6 and 7 system"
      command: /sbin/shutdown -t now
      when: ansible_distribution == "CentOS" and
            (ansible_distribution_major_version == "6" or ansible_distribution_major_version == "7")

A number of Jinja2 `filters`_ can be used in **when** statements.

.. _filters: http://docs.ansible.com/playbooks_filters.html

Suppose we want to ignore the error of one statement and then decide to do something conditionaly based on success or failure::

  tasks:
    - command: /bin/false
      register: result
      ignore_errors: True

    - command: /bin/something
      when: result|failed
    - command: /bin/something_else
      when: result|success
    - command: /bin/still/something_else
      when: result|skipped

If you get some variable that's a string and you'll do a matn comparison on it::

  tasks:
    - shell: echo "only on RedHat 6, derivatives, and later"
      when: ansible_os_family == "RedHat" and ansible_lsb.major_release|int >= 6

.. note::

  The above example requires the lsb_release package on the target host in order to return the ansible_lsb.major_release fact.


::
  
  tasks:
    - command: echo {{item}}
      with_items: [1, 2, 3, 4, 5]
      when: item > 3

When combining **when** with **with_items** , be aware that the **when** statement is processed separately for each item.

Applying 'When' to roles and includes
========================================

Note that if you have several tasks that all share the same conditional statement, you can affix the conditional to a task include statement as below. Note this does not work with playbook includes, just task includes. All the tasks get evaluated, but the conditional is applied to each and every task::

  - include: tasks/sometasks.yml
    when: ansible_os_family == "RedHat"


  - hosts: webservers
    roles:
      - { role: debian_stock_config, when: ansible_os_family == "Debian" }

Conditional Imports
======================

Here is an example::

  ---
  - hosts: all
    remote_user: root
    var_files:
      - "vars/common.yml"
      - [ "vars/{{ansible_os_family}}.yml", "vars/os_defaults.yml" ]

    tasks:
      - name: make sure apache is running
        service: name={{ apache }} state=running

.. note::

  Ansible's approach to configuration - separating variables from tasks, keeps your playbooks from turning into arbitrary code with ugly nested ifs, conditionals, and so on - and results in more streamlines & auditable configuration rules - especially because there are a minimum of decision points to track.

Selecting Files and Templates Based On Variables
===================================================

::

  - name: template a file
    template: src={{item}} dest=/etc/apps/foo.conf
    with_first_found:
      - files:
          - {{ansible_os_family}}.conf
          - defaults.conf
        path:
          - search_path_one/somedir/
          - /opt/other/dir/

.. note::

  Sometimes a configuration file you want to copy, or a template you will use may depend on a variable. This example construct selects the first available file appropriate for the variables of a given host, which is often much cleaner than putting a lot of if conditionals in a template.


Register Variables
=======================

The *register* keyword decides what variable to save a result in. The resulting can be used in templates, action lines, or *when* statements.

::

  ---
  - hosts: all
    remote_user: root
    gather_facts: true

    tasks:
      - name: register results
        command: cat /run/mysqld/mysqld.pid
        register: result

      - name: result failed
        service: name=mysqld state=running
        when: result|failed

      - name: result success
        shell: cat /run/mysqld/mysqld.pid > /root/logfile
        when: result.stdout != ''

As result quite farmiliar with:: 

  {u'changed': True, u'end': u'2015-06-10 12:51:43.636894', u'stdout': u'1351', u'cmd': [u'cat', u'/run/sshd.pid'], u'start': u'2015-06-10 12:51:43.630985', u'delta': u'0:00:00.005909', u'stderr': u'', u'rc': 0, 'invocation': {'module_name': 'command', 'module_args': 'cat /run/sshd.pid'}, 'stdout_lines': [u'1351']}

The registered result can be used in the *with_item* of a task if it is converted into a list as shown below::

  tasks:
    - name: get home dirs
      command: ls /home
      register: home_dirs

    - name: backup all home dirs
      file: path=/mnt/backup/{{item}} src=/home/{{item}} state=link
      with_items: home_dirs.stdout_lines
         # same as with_items: home_dirs.stdout.split()
