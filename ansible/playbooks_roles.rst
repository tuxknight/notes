.. _playbook_roles:

===============
Playbook Roles
===============

.. contents:: Topics

Task Include Files And Encouraging Reuse
==============================================

The goal of a play in a playbook is to map a group of systems into multiple roles::

  ---
  # possibly saved as tasks/foo.yml
  - name: foo
    command: /bin/foo
  - name: bar
    command: /bin/bar

  ---
  tasks:
    - include: tasks/foo.yml

You can also pass variables into includes::

  tasks:
    - include: wordpress.yml wp_user=timmy
    - include: wordpress.yml wp_user=alice
    - include: wordpress.yml wp_user=bob

Includes can also be used in the 'handlers' section::

  ---
  # handlers/handlers.yml
  - name: restart apache
    serice: name=apache state=restarted

  ---
  # in your main playbook file
  handlers:
    - include: handlers/handlers.yml

Roles
=======

Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Grouping content by roles also allows easy sharing of roles with other users.

Example project structure::

  site.yml
  webservers.yml
  fooservers.yml
  roles/
     common/
       files/
       templates/
       tasks/
       handlers/
       vars/
       defaults/
       meta/
     webservers/
       files/
       templates/
       tasks/
       handlers/
       vars/
       defaults/
       meta/

In a  playbook, it would look like this::

  ---
  - hosts: webservers
    roles: 
      - common
      - webservers
      - { role: foo_app_instance, dir: '/opt/a', port: 5000 }
      - { role: some_role, when: "ansible_os_family == 'RedHat'" }
      - { role: foo, tags: ["bar", "baz"] }

main.yml in tasks/ handlers/ vars/ will all be added to the play, and main.yml in meta/ will add any role dependencies to the list of roles.

copy tasks, script tasks can reference files in roles/xxx/files/ without have to path them relatively or absolutely.

template tasks can reference files in roles/xxx/templates/ without have to path them relatively or absolutely.

include tasks can reference files in roles/x/tasks/ without having to path them relatively or absolutely.

Role Default Variables
=======================

Role default variables allow you to set default variables for included or dependent roles (see below). To create defaults, simply add a defaults/main.yml file in your role directory. These variables will have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

Role Dependencies
===================

Role dependencies are stored in the meta/main.yml file contained within the role directory::

  ---
  dependencies:
    - { role: common, some_parameter: 3 }
    - { role: apache, port: 80 }
    - { role: postgres, dbname: blarg, other_parameter: 12 }
    - { role: '/path/to/common/roles/foo', x: 1 }

Roles dependencies are always executed before the role that includes them, and are recursive. 

By default, roles can also only be added as a dependency once - if another role also lists it as a dependency it will not be run again. This behavior can be overridden by adding allow_duplicates: yes to the meta/main.yml file. 

Embedding Modules In Roles
=============================

You may distribute a custom module as part of a role. You could add a directory named 'library' which include the module directly inside of it. And the module will be usable in the role itselft, as well as any roles that are called after this role.
