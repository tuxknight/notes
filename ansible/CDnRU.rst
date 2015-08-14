.. _CDnRU:

==========================================
Continuous Delivery and Rolling Upgrades
==========================================

This document describes how to achieve the continuous delivery and zero-downtime updates, using one of Ansible's most complete example playbooks as a template: `lamp_haproxy`_ .

.. _lamp_haproxy: https://github.com/ansible/ansible-examples/tree/master/lamp_haproxy

This playbook deploy Apache, PHP, MySQL, Nagios and HAProxy to a CentOS-based set of servers.

Site Deployment
================

Let us start with ``site.yml``

::

  ---
  # This playbook deploys the whole application stack in this site.
  
  # Apply common configuration to all hosts
  - hosts: all
  
    roles:
    - common
  
  # Configure and deploy database servers.
  - hosts: dbservers
  
    roles:
    - db
  
  # Configure and deploy the web servers. Note that we include two roles
  # here, the 'base-apache' role which simply sets up Apache, and 'web'
  # which includes our example web application.
  
  - hosts: webservers
  
    roles:
    - base-apache
    - web
  
  # Configure and deploy the load balancer(s).
  - hosts: lbservers
  
    roles:
    - haproxy
  
  # Configure and deploy the Nagios monitoring node(s).
  - hosts: monitoring
  
    roles:
    - base-apache
    - nagios
  
In this playbook we have 5 plays. The first one targets ``all`` hosts and applies the ``common`` role to all of the hosts. This is for site-wide things like yum repository configuration, firewall configuration, and anything else that needs to apply to all of the servers.

The next four plays run against specific host groups and apply specific roles to those servers. Along with the roles for Nagios monitoring, the database, and the web application, we’ve implemented a ``base-apache`` role that installs and configures a basic Apache setup. This is used by both the sample web application and the Nagios hosts.

Reusable Content: Roles
==========================

Roles are a way to organize content: tasks, handlers, templates, and files, into reusable components.

This example has six roles: ``common`` , ``base-apache`` , ``db`` , ``haproxy`` , ``nagios`` , and ``web`` . Most sites will have one or more common roles that are applied to all systems, and then a series of application-specific roles that install and configure particular parts of the site.

Roles can have variables and dependencies and you can pass in parameters to roles to modify their behavior.

Configuration: Group Variables
================================

Group variables are variables that are applied to groups of servers. They can be used in templates and in playbooks to customized behavior and to provide easily-changed settings and parameters.

They are stored in a directory called ``group_vars`` in the same location as your inventory. These variables are used in a variety of places. You can use them in playbooks, templates.

The Rolling Upgrade
=====================

``rolling_upgrade.yml`` was made up of two plays.

::

  - hosts: monitoring
    tasks: []

This play forces a fact-gathering step on our monitoring servers.

The next part is the update play which first part looks like this::

  - hosts: webservers
    user: root
    serial: 1

This is a play definition, operating on the ``webservers`` group. The ``serial`` keyword tells ansible how many servers to operate on at once.If it is not specified, Ansible will parallelize these operations up to the default 'forks' limit specified in the configuration file. But for a zero-downtime rolling upgrade, you may not want to operate on that many hosts at once.

Here is the next part of the update play::

  pre_tasks:
  - name: disable nagios alerts for this host webserver service
    nagios: action=disable_alerts host={{inventory_hostname}} services=webserver
    delegate_to: "{{item}}"
    with_items: groups.monitoring
  
  - name: disable the server in haproxy
    shell: echo "disable server myapplb/{{inventory_hostname}} "| socat stdio /var/lib/haproxy/stats
    delegate_to: "{{item}}"
    with_items: groups.lbservers

The ``pre_tasks`` keyword just lets you list tasks to run before the roles are called. The ``delegate_to`` and ``with_items`` arguments, used together, cause Ansible to loop over each monitoring server and load balancer, and perform that operation(delegate that operation) on the monitor or load balancing server, "on behalf" of the webserver.

In programming terms, the outer loop is the list of web servers, and the inner loop is the list of monitoring servers.

In the ``post_tasks`` section, we reverse the changes to the Nagios configuration and put the web server back in the load balancing pool::

  post_tasks:
  - name: Enable the server in haproxy
    shell: echo "enable server myapplb/{{inventory_hostname}}"| socat stdio /var/lib/haproxy/stats
    delegate_to: "{{item}}"
    with_items: groups.lbservers
  - name: re-enable nagios alerts
    nagios: action=enable_alerts host={{inventory_hostname}} services=webserver
    delegate_to: groups.monitoring

Continuous Delivery End-To-End
=================================

Now that you have an automated way to deploy updates to your application, how do you tie it all together? A lot of organizations use a continuous integration tool like `Jenkins`_ or `Atlassian Bamboo`_ to tie the development, test, release, and deploy steps together. You may also want to use a tool like `Gerrit`_ to add a code review step to commits to either the application code itself, or to your Ansible playbooks, or both.

.. _Jenkins: http://jenkins-ci.org/
.. _Atlassian Bamboo: https://www.atlassian.com/software/bamboo
.. _Gerrit: https://code.google.com/p/gerrit/

Depending on your environment, you might be deploying continuously to a test environment, running an integration test battery against that environment, and then deploying automatically into production. Or you could keep it simple and just use the rolling-update for on-demand deployment into test or production specifically. This is all up to you.

This should give you a good idea of how to structure a multi-tier application with Ansible, and orchestrate operations upon that app, with the eventual goal of continuous delivery to your customers. You could extend the idea of the rolling upgrade to lots of different parts of the app; maybe add front-end web servers along with application servers, for instance, or replace the SQL database with something like MongoDB or Riak. Ansible gives you the capability to easily manage complicated environments and automate common operations.

