.. _playbooks_loops:

=======
Loops
=======

.. contents:: Topics

Standard Loops
================

::

  - name: common packages
    apt: name={{item.name}} state={{item.state}}
    with_items:
      - {name:"vim-enhanced", state:"present"}
      - {name:"build-essentials", state:"present"}
      - {name:"firefox", state:"abent"}

Just to save some typing of repeat sections, we use *loop* statements. It works fine with a YAML list in a variables file or the vars section.

Nested Loops
==============

::

  - name: make three copies of original file
    copy: src={{item[0]}} dest={{item[1]}}
    with_nested:
      - ["foo.conf", "bar.conf"]
      - ["/etc/app.d/", "/usr/share/app.d/, "/usr/local/app.d"]

Looping over Hashed
=====================

Suppose you have the following variable:

::

  ---
  users:
    alice:
      name: Alice Brown
      tele: 32323232
    bob:
      name: Bob Green
      tele: 1010100101

You can loop through the elements of a hash using ``with_dict`` like this::

  tasks:
  - name: print info
    debug: msg="User {{item.key}} is {{item.value.name}} {{item.value.tele}}"
    with_dict: "{{users}}"

Looping with Fileglobs
=======================

*with_fileglobs* matched all files in a single directory, non-recursively, that match a pattern::

  ---
  - hosts: apps
    tasks:
      - file: dest=/etc/apps state=directory
      - copy: src={{item}} dest=/etc/apps owner=root mode=600
        with_fileglob:
          - /playbooks/files/apps/*.conf

.. note::

  When using a relative path with *with_fileglob* in a role, Ansible resolves the path relative to the roles/<rolename>/files directory

Looping over Integer Sequences
================================

::

  tasks:
  - name: sequence loops
    file: dest=/tmp/loops.{{item}} state=touch
    with_sequence: start=4 end=23 stride=3


Random Choices
=================

Random Choices is not a good load balancer approach. It can be used to add chaos and excitement to otherwise predictable automation environments::

  tasks:
    - name: delete files by random choice
      file: src={{item}} state=absent
      with_random_choice:
        - "/root/.bashrc"
        - "/etc/default/grub"
        - "/var/log/message"

Do-Until Loops
================

::

  tasks:
    - action: shell /usr/bin/foo
      register: result
      until: result.stdout.find("all systems go") != -1
      retries: 5
      delay: 10

Do-Until loops would retry a task untill acertain condition is met.
