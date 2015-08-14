.. _yaml_syntax:

==============
YAML Syntax
==============

.. contents:: Topics

Basics
==========

All YAML files should begin with ``---`` .

All members of a list are lines beginning at the same indentation level starting with a ``-`` (a dash and a space)::

  ---
  # A list of tasty fruits
  - Apple
  - Orange
  - Strawberry
  - Mango

Dictionaries can also be represented in an abbreviated form if you really want to::

  ---
  { name: MEM, job: Dev, skill: delta }

Ansible doesnt really use these too much, but you can also specify a boolean value(true/false) in several forms::

  ---
  create_key: yes
  needs_agent: no
  knows_oop: True
  likes_emacs: TRUE
  uses_cvs: false

Gotchas
========

While YAML is generally friendly, the following is going to result in a YAML syntax error::

  foo: somebody said I should put a colon here: so I did

You wil want to quote any hash values using colons::

  foo: "someone said i should put a colon: so i did"

Further, Ansible uses "{{var}}" for variables. If a value after a colon starts with a "{" , YAML will think it is a dictionary, so you must quote it::

  foo: "{{ varialbe }}"
