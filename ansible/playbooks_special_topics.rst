.. _playbooks_special_topics:

==========================
Playbooks Special Topics
==========================

.. contents:: Topics

Ansible Privilege Escalation
==============================

directives
--------------

become
  equivalent to adding 'sudo:' or 'su:' to a play or task

become_user
  equivalent to adding 'sudo_user:' or 'su_user:' to a play or task

become_method
  at play or task level overrides the default method set in ansible.cfg, set to 'sudo' 'su' 'pbrun' 'pfexec'

ansible variables
-------------------

ansible_become
  equivalent to ansible_sudo or ansible_su, allows to force privilege escalation

ansible_become_method
  allows to set privilege escalation method

ansible_become_user
  equivalent to ansible_sudo_user or ansible_su_user, allows to set the user you become through privilege escalation

ansible_become_pass
  equivalent to ansible_sudo_pass or ansible_su_pass, allows you to set the privilege escalation password

command line options
----------------------

--ask-become-pass
  ask for privilege escalation password

-b, --become
  run operation with become

--become-method=BECOME_METHOD
  privilege escalation method to use(default=sudo) choices:[sudo|su|pbrunpfexec]

--become-user=BECOME_USER
  run operations as this user(default=root)



Asynchronous Action and Polling
=================================

By default tasks in playbooks block, meaning the connections stay open until task is done on each node. But you may be running operations that take longer than the SSH timeout.

To launch a task asynchronously, specify its maximum runtime and how frequently you would like to poll for status::

  ---
  - hosts: all
    remote_user: root
    tasks:
     - name: run a command that costs a very long time
       command: /bin/sleep 30
       async: 45
       poll: 5

.. note::

  There is no default for the async time limit. If you leave off the 'async' keyword, the task runs asynchronously. If you dont need to wait on the task to complete, you my "fire and forget" by specifying a poll value of 0.

If you would like to perform a variation of the "fire and forget" where you "fire and forget, check on it later" you can perform a task similar to the following::

  ---
  - hosts: localhost
    remote_user: root
    tasks:
     - name: run a command that costs a very long time
       apt: name=docker.io state=installed
       async: 40
       poll: 3
       register: result
  
     - name: result
       debug: msg={{result.ansible_job_id}}
  
     - name: jobid
       async_status: jid={{result.ansible_job_id}}
       register: jobresult
       until: jobresult.finished
       retries: 30
       delay: 2
  
     - name: async mesg
       debug: msg={{jobresult}}

Check Mode
===========

When ansible-playbook is executed with ``--check`` it will not take any changes on remote systems. Any module instrumented to support 'check mode' will report what changes they would have made rather that make them.

Other modules that do not support check mode will also take no action.

Check mode is just a simulation, and if you have steps that use conditionals that depend on the result of prior command, it may be less useful for you.

Somtetimes you may want to have a task to be executed even in check mode. Use the ``always_run`` clause on the task::

  tasks:
    - name: this task is run even in check mode
      command: /something/need/to/run/even/in/check/mode
      always_run: yes

A task with a ``when`` clause evaluated to false, will still be skipped even if it has a ``always_run`` clause evaluated to true.

Error Handling In Playbooks
=============================

Generally playbooks will stop executing any more steps on a host that has a failure. Somtetimes, you want to continue on::

  - name: this will not be counted as a failure
    command: /bin/false
    ignore_errors: yes

Handlers and Failure
----------------------

When a task fails on a host, handlers which were previously notified will not be run on that host. This can lead to cases where an unrelated failure can leave a host in an unexpected state. 

For example, a task could update a configuration file and notify a handler to restart some service. If a task later on in the same play fails, the service will not be restarted despite the configuration change.

You can change this behavior with the ``--force-handlers`` command-line option, or by including ``force_handlers: True`` in a play, or ``force_handlers = True`` in ansible.cfg. 

When handlers are forced, they will run when notified even if a task fails on that host. (Note that certain errors could still prevent the handler from running, such as a host becoming unreachable.)

Controlling What Defines Failure
-----------------------------------

Suppose the error code of a command is meaningless and to tell if there is a failure what really matters is the output of the command, for instance if the string "FAILED" is in the output.

::

  - name: this command prints FAILED when it fails
    command: /usr/bin/df -h
    register: r
    failed_when: "'100' in r.stdout"

Overriding The Changed Result
-------------------------------

Sometimes you will know, based on the return code or output that it did not make any changes, and wish to override the “changed” result such that it does not appear in report output or does not cause handlers to fire::

  tasks:
    - name: this will never report 'changed' status
      command: file /bin/sh
      changed_when: False

    - name: this will report 'changed' status 
      command: toucn /var/log/event.log
      register: r
      ignore_errors: true
      changed_when: "r.rc!=2"

Lookups
=========

Lookup plugins allow access of data in Ansible from outside sources. These plugins are evaluated on the Ansible control machine, and can include reading the filesystem but also contacting external datastores and services. These values are then made available using the standard templating system in Ansible, and are typically used to load variables or templates with information from those systems.

File Lookup
-------------

Contents can be read off the filesystem as follows::

  - hosts: all
    vars:
      contents: "{{lookup('file', '/etc/foo.txt')}}"

    tasks:
      - debug: msg="the value of foo.txt is {{contents}}"

.. note:

  Lookups occur on the local computer, not on the remote computer.

Password Lookup
----------------

``password`` generates a random plaintext password and stores it in a file at a given filepath.

More Details will be found as `The Password Lookup`_

.. _`The Password Lookup`: http://docs.ansible.com/playbooks_lookups.html#id5

The CSV File Lookup
--------------------

``csvfile`` lookup reads the contents of a file in CSV format. The lookup ollks for the row where the first column matches ``keyname`` , and returns the value in the first column.

The ``csvfile`` lookup supports several arguments::

  lookup('csvfile', 'key arg1=val1 arg2=val2 ...')

The first value in the argument is the ``key`` , which must be an entry that appears exactly once in column 0 of the table. All other arguments are optional.

+----------+----------------+----------------------------------------------+
|Field     |Default         |Description                                   |
+==========+================+==============================================+
|file      |ansible.csv     |Name of the file to load                      |
+----------+----------------+----------------------------------------------+
|delimiter |TAB             |Delimiter used by CSV file                    |
+----------+----------------+----------------------------------------------+
|col       |1               |The column to output, indexed by 0            |
+----------+----------------+----------------------------------------------+
|default   |empty string    |return value if the key is not in the csv file|
+----------+----------------+----------------------------------------------+

More Lookups
--------------

Here are more examples::

  ---
  - hosts: all
    tasks:
      - debug: msg="{{lookup('env','HOME')}} is and environment varialbe"

      - debug: msg="{{item}}" is a line from the result of this command"
        with_lines:
          -cat /etc/motd
      - debug: msg="{{lookup('pipe','date'}} is the raw result of running this command"
      # redis_kv lookup requires the Python redis package
      - debug: msg="{{lookup('redis_kv','redis://localhost:6379,somekey')}} is value is Redis for somekey"

      # dnstxt lookup requires the Python dnspython package
      - debug: msg="{{ lookup('dnstxt', 'example.com') }} is a DNS TXT record for example.com"

      - debug: msg="{{ lookup('template', './some_template.j2') }} is a value from evaluation of this template"

      - debug: msg="{{ lookup('etcd', 'foo') }} is a value from a locally running etcd"

      # The following lookups were added in 1.9
      - debug: msg="{{item}}"
        with_url:
             - 'http://github.com/gremlin.keys'

      # outputs the cartesian product of the supplied lists
      - debug: msg="{{item}}"
        with_cartesian:
             - list1
             - list2
             - list3

Delegation, Rolling Updates, and Local Actions
================================================

Rolling Update Batch Size
----------------------------

For a rolling updates use case, you can define how many hosts Ansible should manage at a single time by using the ``serial`` keyword::

  - name: rolling updates
    hosts: webservers
    serial: 3
    # or serial: "30%"

And ``serial`` keyword can also be specified as a percentage.

Maximum Failure Percentage
----------------------------

It may be desirable to abort the play when a certain threshold of failures have been reached. To achieve this, you can set a maximum failure percentage on a play as follows::

  - hosts: webservers
    max_fail_percentage: 30
    serial: 10

If more than 3 of the 10 servers in the group were to fail, the rest of the play would be aborted.

.. note::

  The percentage set must be exceeded, not equaled. For example, if serial were set to 4 and you wanted the task to abort when 2 of the systems failed, the percentage should be set at 49 rather than 50.

Delegation
-------------

If you want to perform a task on one host with reference to other hosts, use the ``delegate_to`` keyword on a task. Using this with the ``serial`` keyword to control the numbers of hosts executing at one time is also a good idea::

  ---
  - hosts: webservers
    serial: 5
    tasks:
      - name: task out of load balancer pool
        command: /usr/bin/task_out_of_pool {{inventory_hostname}}
        delegate_to: 127.0.0.1

      - name: actual steps would go here
        yum: name=acme-web-stack state=latest

      - name: add back to load balancer pool
        command: /usr/bin/add_back_to_pool {{inventory_hostname}}
        delegate_to: 127.0.0.1

There is also a shorthand syntax that you can use on e per-task basis: ``local_action`` . A common pattern is to use a local action to call ``rsync`` to recursively copy files to the managed servers::

  ---
  # ...
    tasks:
      - name: recursively copy file from management server to target
        local_action: command rsync -a /path/to/files {{inventory_hostname}}:/path/to/target/

Run Once
-----------

It can be achieved by configuring ``run_once`` on a task to only run a task one time and only on one host.

::

  ---
  #...
    tasks:
      - name: run once
        command: /opt/app/update_db.py
        run_once: true

This can be optionally paired with ``delegate_to`` to specify an individual host to execute on. When ``run_once`` is not used with ``delegate_to`` it will execute on the first host::

  ---
  #...
    tasks:
      - name: run once
        command: /opt/app/update_db.py
        run_once: true
        delegate_to: web1.app.com

Local Playbooks
----------------

To run an entire playbook locally rather than by connecting over SSH, just set the ``hosts:`` line to ``hosts: 127.0.0.1`` and then run the playbook like so::

  ansible-playbook playbook.yml --connection=local

Alternatively, a local connection can be used in a single playbook play::

  - hosts: 127.0.0.1
    connection: local

Setting the Environment
=========================

It is easy to configure your environment by using the ``environment`` keyword::

  - hosts: all
    remote_user: root
    tasks:
      - apt: name=cobble state=installed
        environment:
          http_proxy: http://proxy.example.com:8080

The environment can also be stored in a variable, and accessed like so::

  - hosts: all
    remote_user: root
    vars:
      proxy_env:
        http_proxy: http://proxy.example.com:8080

    tasks:
      - apt: name=cobble state=installed
        environment: proxy_env

The most logical place to define an environment hash might be a group_vars file::

  ---
  # file: group_vars/hf
  net_server: ntp.hf.example.com
  backup: bak.hf.example.com
  proxy_env:
    http_proxy: http://proxy.hf.example.com:8080

Prompts
===========

When running a playbook, you may wish to prompt the user for certain input, and can do so with the ``vars_prompt`` section::

  ---
  - hosts: all
    remote_user: root
    vars_prompt:
      name: "what is your name? "
      quest: "what is your quest? "

You can set a default argument::

  ---
  - hosts: all
    remote_user: root
    vars_prompt:
      - name: "release_version"
        prompt: "Input Version"
        default: "1.0"

An alternative form of ``vars_prompt`` allows for hiding input from the user::

  vars_prompt:
    - name: "u_pwd"
      prompt: "Enter your password"
      private: yes

If ``passlib`` is installed, ``vars_prompt`` can also crypt the entered value so you can use it to define a password::

  vars_prompt:
    - name: "passwd"
      prompt: "Input Your Password"
      private: yes
      confirm: yes
      encrypt: "sha512_crypt"
      salt_size: 7

You can use your own salt using ``salt`` , or have one generated automatically using ``salt_size`` . If nothing is specified, a salt of size 8 will be generated.

Here are some crypt scheme supported by ``passlib``::

    * des_crypt - DES Crypt
    * bsdi_crypt - BSDi Crypt
    * bigcrypt - BigCrypt
    * crypt16 - Crypt16
    * md5_crypt - MD5 Crypt
    * bcrypt - BCrypt
    * sha1_crypt - SHA-1 Crypt
    * sun_md5_crypt - Sun MD5 Crypt
    * sha256_crypt - SHA-256 Crypt
    * sha512_crypt - SHA-512 Crypt
    * apr_md5_crypt - Apache’s MD5-Crypt variant
    * phpass - PHPass’ Portable Hash
    * pbkdf2_digest - Generic PBKDF2 Hashes
    * cta_pbkdf2_sha1 - Cryptacular’s PBKDF2 hash
    * dlitz_pbkdf2_sha1 - Dwayne Litzenberger’s PBKDF2 hash
    * scram - SCRAM Hash
    * bsd_nthash - FreeBSD’s MCF-compatible nthash encoding

Tags
=====

If you have a large playbook it may become useful to be able to run a specific part of the configuration without running the whole playbook.

Both plays and tasks support a "tags:" attribute for this reason::

  tasks:
    - yum: name="{{item}}" state=installed
      with_items:
        - httpd
        - memcached
      tags:
        - packages

    - template: src=templates/src.j2 dest=/etc/foo.conf
      tags:
        - configuration


If you wanted to just run the "configuration" and "packages" part of a very long playbook, you could do this::

  ansible-playbook site.yml --tags "configuration,packages"

On the other hand, if you want to run a playbook *without* certain tasks, you would do this::

  ansible-playbook site.yml --skip-tags "notification"

You may also apply tags to roles::

  roles:
    - {role: webserver, port: 5000, tags:['web','foo']}

And you may also tag basic include statements::

  - include: foo.yml tags=web,foo

There is a special ``always`` tag that will always run a task, unless specifically skipped (--skip-tags always).

::

  tasks:
    - debug: msg="Always runs"
      tags:
        - always
    - debug: msg="runs when you use tag1"
      tags:
        - tag1

There are another 3 special keywords for tags ``tagged`` ``untagged`` ``all``

::

  ansible-playbook site.yml --tags all
  ansible-playbook site.yml --tags tagged
  ansible-playbook site.yml --tags untagged

Start and Step
===============

If you want to start executing your playbook at a particular task, you can do so with the ``--start-at-task`` option::

  ansible-playbook playbook.yml --start-at-task="install packages"

The above will start executing your playbook at a task named "install packages"

Playbooks can also be executed interactively with ``--step`` ::

  ansible-playbook playbook.yml --step

Answering 'y' will execute the task , 'n' will skip the task and 'c' will continue executing all the remaining tasks without asking.
