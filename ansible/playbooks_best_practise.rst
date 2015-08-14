.. _playbook_best_practise:

==========================
Playbook Best Practises
==========================

.. contents:: Topics

Content Organization
========================

`Roles`_ are great!

.. _Roles: http://docs.ansible.com/playbooks_roles.html

Directory Layout
------------------

The top level of the directory would contain files and directories like so::

  production                # inventory file for production servers
  stage                     # inventory file for stage environment
  
  group_vars/
     group1                 # here we assign variables to particular groups
     group2                 # ""
  host_vars/
     hostname1              # if systems need specific variables, put them here
     hostname2              # ""
  
  library/                  # if any custom modules, put them here (optional)
  filter_plugins/           # if any custom filter plugins, put them here (optional)
  
  site.yml                  # master playbook
  webservers.yml            # playbook for webserver tier
  dbservers.yml             # playbook for dbserver tier
  
  roles/
      common/               # this hierarchy represents a "role"
          tasks/            #
              main.yml      #  <-- tasks file can include smaller files if warranted
          handlers/         #
              main.yml      #  <-- handlers file
          templates/        #  <-- files for use with the template resource
              ntp.conf.j2   #  <------- templates end in .j2
          files/            #
              bar.txt       #  <-- files for use with the copy resource
              foo.sh        #  <-- script files for use with the script resource
          vars/             #
              main.yml      #  <-- variables associated with this role
          defaults/         #
              main.yml      #  <-- default lower priority variables for this role
          meta/             #
              main.yml      #  <-- role dependencies
  
      webtier/              # same kind of structure as "common" was above, done for the webtier role
      monitoring/           # ""
      fooapp/               # ""

Use Dynamic Inventory With Clouds
-------------------------------------

If you are using a cloud provider, you should not be managing your inventory in a static file.  `See Dynamic Inventory`_

.. _See Dynamic Inventory: http://docs.ansible.com/intro_dynamic_inventory.html

If you have another system maintaining a canonical list of systems in your infrastructure, usage of dynamic inventory is a great idea in general.

How to Differentiate Stage vs Production
------------------------------------------

It is suggested that you define groups based on purpose of the hosts and also geography or datacenter location::

  # file: production
  
  [atlanta-webservers]
  www-atl-1.example.com
  www-atl-2.example.com
  
  [boston-webservers]
  www-bos-1.example.com
  www-bos-2.example.com
  
  [atlanta-dbservers]
  db-atl-1.example.com
  db-atl-2.example.com
  
  [boston-dbservers]
  db-bos-1.example.com
  
  # webservers in all geos
  [webservers:children]
  atlanta-webservers
  boston-webservers
  
  # dbservers in all geos
  [dbservers:children]
  atlanta-dbservers
  boston-dbservers
  
  # everything in the atlanta geo
  [atlanta:children]
  atlanta-webservers
  atlanta-dbservers
  
  # everything in the boston geo
  [boston:children]
  boston-webservers
  boston-dbservers

Group And Host Variables
----------------------------

Group Variables and Host Variables will set different values to different groups or hosts.

For instance, atlanta-webservers have different http port with boston-webservers::

  ---
  # file: host_vars/atlanta-webservers
  http_port: 8080
  ---
  # file: host_vars/boston-webservers
  http_port: 80

Top Level Playbooks Are Separated By Role
-------------------------------------------

In site.yml, we include a playbook that defines our entire infrastructure, just including some other playbooks::

  ---
  # file: site.yml
  - include: webservers.yml
  - include: dbservers.yml

In a file like webservers.yml, we simplay map the configuration of the webservers group to the roles performed by the webservers group::

  ---
  # file: webservers.yml
  - hosts: webservers
    roles: 
      - common
      - webtier

The idea here is that we can choose to configure our whole infrastructure by running site.yml, or we could choose to run a subset by running webservers.yml::

  ansible-playbook site.yml --limit webservers
  ansible-playbook webservers.yml

Task And Handler Organization For A Role
------------------------------------------

Here is an example tasks file that explains how a role works::

  ---
  # file: roles/common/tasks/main.yml
  
  - name: be sure ntp is installed
    yum: pkg=ntp state=installed
    tags: ntp
  
  - name: be sure ntp is configured
    template: src=ntp.conf.j2 dest=/etc/ntp.conf
    notify:
      - restart ntpd
    tags: ntp
  
  - name: be sure ntpd is running and enabled
    service: name=ntpd state=running enabled=yes
    tags: ntp

Role *common* just sets up NTP, Handlers are only fired when certain tasks report changes, and are run at the end of each play::

  ---
  # file: roles/common/handlers/main.yml
  - name: restart ntpd
    service: name=ntpd state=restarted

Always Mention The State
===========================

The 'state' parameter is optional to a lot of modules. Whether 'state=present' or 'state=absent', it's always best to leave that parameter in your playbooks to make it clear.

Group By Roles
================

This allows playbooks to target machines based on role, as well as to assign role specific variables using the group variable system.

Operating System and Distribution Variance
============================================

When dealing with a parameter that is different between two different operating systems, a great way to handle this is by using the group_by module::

  ---
  #talk to all hosts just so we can learn about them
  - hosts: all
    tasks: 
      - group_by: key=os_{{ ansible_distribution }}
  
  - hosts: os_CentOS
    gather_facts: False
    tasks:
      - # tasks that only happen on CentOS go here

This will throw all systems into a dynamic group based on the os name.

If group-specific settings are needed, this can also be done. For example::

  ---
  # file: group_vars/all
  applist: 20
  
  ---
  # file: group_vars/os_CentOS
  applist: 10

Bundling Ansible Modules With Playbooks
=========================================

If a playbook has a './library' directory relative to its YAML file, this directory can be used to add ansible modules that will automatically be in the ansible module path.

Whitespace and Comments
=========================

Generous use of whitespace to break things up, and use of comments (which start with ‘#’), is encouraged.

Always Name Tasks
===================

This name is shown when the playbook is run.

Keep It Simple
================

If something feels complicated, it probably is, and may be a good opportunity to simplify things.

Version Control
================

Use version control.
