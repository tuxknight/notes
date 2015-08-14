
====================
Dynamic Inventory
====================

.. contents:: Topics

Ansible provides a basic text-based system as described in :ref:`inventory_intro`. And it easily supports all of these options via an external invnetory system.

Example. AWS EC2 External Inventory Script
============================================

You can use this `script`_ in one of two ways. The easiest is to use Ansible's ``-i`` command line option and specify the path to the script after marking it executable::

  ansible -i ec2.py -u ubuntu us-east-1d -m ping

The second option is to copy the script to ``/etc/ansible/hosts`` and ``chmod +x`` it. You will also need to copy the `ec2.ini`_ file to ``/etc/ansible/ec2.ini``

.. _`script`: https://raw.github.com/ansible/ansible/devel/plugins/inventory/ec2.py
.. _`ec2.ini`: https://raw.githubusercontent.com/ansible/ansible/devel/plugins/inventory/ec2.ini

Using Multiple Inventory Sources
==================================

If the location given to ``-i`` in Ansible is a directory, Ansible can use multiple inventory sources at the same time.

Static Groups of Dynamic Groups
=================================

When defining a static group of dynamic child groups, define the dynamic groups as empty in the static inventory file::

  [foo]

  [bar]

  [cluster:children]
  foo
  bar


========================================
Developing Dynamic Inventory Sources
========================================

You just need to create a script or program that can return JSON in the right format when fed the proper arguments.

Script Conventions
====================

When the external node script is called with the single argument ``--list`` , the script must return a JSON dictionary of all the groups to be managed.

::

  {
    "databases":{
      "hosts": ["host01", "host02"],
      "vars": {
        "foo": bar,
        "flag": true
      }
    },
    "webservers":["web01", "web02"],
    "apps":{
      "hosts":["backend01", "backend02"],
      "vars":{
        "local": false
      },
      "children":["databases","webservers"]
    }
  }

When called with the arguments ``--host <hostname>`` , the script must return either an empty JSON dictionary, or a dictionary of variables to make available to templates and playbooks.
