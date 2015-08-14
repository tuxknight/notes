.. _playbooks_variables:

===========
Variables
===========

.. contents:: Topics

Variables in Ansible are how we deal with differences between systems.

What Makes A Valid Variable Name
==================================

Variable names should be letters, numbers and underscores. Variables should always start with a letter.

Variable Defined in Inventory
==============================

See :ref:`inventory_intro` document for multiple ways on how to define variables in inventory.

Varialbe Defined in a Playbook
================================

::

  - hosts: webservers
    vars:
      http_port: 80

In a playbook, varialbes are defined directly inline like above.

Varialbes defined from include files and roles
================================================

Usage of roles is preferred as it provides a nice organizational system. Variables can also be included in the playbook via include files.

Using Variables: About Jinja2
===============================

Ansible allows you to reference variables in your playbooks using the Jinja2 templating system.

This is basic form of variable substituion ``My amp goes to {{ max_amp_value }}`` 

And it's also valid directly in playbooks ``template: src=foo.cfg dest={{ remote_install_path }}/foo.cfg``

YAML Quote
=============

YAML syntax requires that if you start a value with ``{{ foo }}`` you quote the whole line, since it wants to be sure you aren't try to start a YAML dictionary.

This won't work::
  
  - hosts: backend
    vars:
      srv_path: {{ base_path }}/source

Do it like this and you'll be fine::

  - hosts: backend
    vars:
      srv_path: "{{ base_path }}/source"


Information Discoverd From Systems: Facts
===========================================

Facts are information derived from speaking with your remote systems. 

Try it out ``ansible -i hosts all -m setup``  and ``ansible -i hosts all -m setup -a filter=ansible_mounts``

So, facts may be referenced in a template or playbook as ``{{ ansible_devices.sda.model }}``

Facts are frequently used in :ref:`playbooks_conditionals` and also in templates.

Turning Off Facts
===================

::

  - hosts: lbservers
    gather_facts: false

Local Facts(Facts.d)
=======================

If a remotely managed system has an “/etc/ansible/facts.d” directory, any files in this directory ending in ”.fact”, can be JSON, INI, or executable files returning JSON, and these can supply local facts in Ansible.

For instance assume a /etc/ansible/facts.d/preferences.fact::

  [general]
  asdf=1
  bar=2

``ansible localhost -m setup -a "filter=ansible_local"``

You will see the following fact added::

  "ansible_local": {
      "preferences": {
        "general": {
          "asdf": "1",
          "bar": "2"
        }
      }
  }

and this data can be accessed in a template/playbook as::

  {{ ansible_local.preferences.general.asdf }}

Fact Caching
==============

It is possible for one server to reference variables about another, like so::

  {{ hostvars['asdf.example.com']['ansible_os_family'] }}

Ansible must have already talked to 'asdf.example.com' in the current or another play up higher playbook.

To avoid this, ansible allows the ability to save facts between playbook runs. With fact-caching enabled, it would not be necessary to "hit" all servers to reference variables and information about them.

To enable fact-caching in ansible.cfg as follows::

  [defaults]
  gatering = smart
  fact_caching = redis
  fact_caching_timeout = 86400

redis is the only supported fact caching engine::

  yum install redis
  service redis start
  pip install redis

Registered Variables
=====================

Registered varialbes save the result of a command. Use of ``-v`` when executing playbooks will show possible values for the results.

Here's a quick syntax example::

  - hosts: localhost
    tasks:
      - name: add user ansible_test
        user: name=ansible_test state=present shell=/bin/false
        register: user_result
        ignore_errors: True

      - name: copy vimrc files
        copy: src=.vimrc dest=/home/ansible_test/.vimrc
        when: user_result.state == "present"

Accessing Complex Variable Data
=================================

Different ways to access variable data::

  {{ ansible_eth0["ipv4"]["address"] }}
  {{ ansible_eth0.ipv4.address }}
  {{ ansible_eth0[0] }}

Magic Variables, and How TO Access Information About Other Hosts
===================================================================

Ansible provides a few variables for you automatically, mostly like ``hostvars`` ``group_names`` ``groups``

* hostvars
  
  **hostvars** lets you ask about the variables of another host including facts that have been gathered about that host. 

  ::

    {{ hostvars['node1.server.com']['ansible_os_family'] }}

* group_names

  **group_names** is a list of all the groups the current host is in.

  ::

    {% if 'webserver' in group_names %}
      # some part of a configuration file that only applies to webservers
    {% endif %}

* groups

  **groups** is a list of all the groups in the inventory.

  ::

    {% for host in groups['webserver'] %}
      # something that applies to all app servers.
      {{ hostvars[host]['ansible_eth0']['ipv4']['address']}} # this could walk out all IP addresses in a group
    {% endfor %}

* play_hosts

  **play_hosts** is available as a list of hostnames that are in scope for the current play.

* delegate_to

  **delegate_to** is the inventory hostname of the host that the current task has been deledated to using 'delegate_to'.

* inventory_dir && inventory_file
  
  **inventory_dir** is the pathname of the directory holding Ansible's inventory host file.
  
  **inventory_file** is the pathname and the filename pointing to the Ansible's inventory host file.

Variable File Separation
==========================

To keep certain important variables private or just to keep certain information in different files, you can do this using external varialbes files::

  ---
  - hosts: all
    remote_user: root
    vars:
      favcolor: blue
    vars_files:
      - /vars/external_vars.yml

    tasks:
    - name: create a file
      copy: src=/tmp/ansible_test content="{{ external_vars }}"

Using external varialbe files could removes the risk of sharing sensitive data with others when sharing your playbook source with them.

The content of each varialbes file is simple YAML dictionary, like this::

  ---
  somevar: somevalue
  passowrd: sensitive

Passing Variables On The Command Line
=======================================

``ansible-playbook release.yml --extra-vars "hosts=lbs user=tdm"``

::

  ---
  - hosts: '{{ hosts }}'
    remote_user: '{{ user }}'
    tasks:
      - name: update apps
        git: repo=git@github.com/gituser/somerepo.git dest='/home/{{user}}'

Extra vars can be loaded as quoted JSON or from a JSON file with "@"::

  --extra-vars '{"hosts": "lbs", "user": "tdm"}'
  --extra-vars "@some_file.json"

Variable Precedence: Where Should I Put A Variable ?
=======================================================

* extra vars (-e in the command line) always win
* then comes connection variables defined in inventory (ansible_ssh_user, etc)
* then comes "most everything else" (command line switches, vars in play, included vars, role vars, etc)
* then comes the rest of the variables defined in inventory
* then comes facts discovered about a system
* then "role defaults", which are the most "defaulty" and lose in priority to everything.

Site wide defaults should be defined as a 'group_vars/all' setting.

Official document provides some `examples`_

.. _examples: http://docs.ansible.com/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable
