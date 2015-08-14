.. _playbooks_filters:

===================
Playbooks Filters
===================

.. contents:: Topics

Filters For Formatting Data
=============================

The following filters will take a data structure in a template and render it in a slightly different format. These are occasionally useful for debugging::

  {{some_varialbe|to_nice_json}}
  {{some_varialbe|to_nice_yaml}}

Filters Often Used With Conditionals
======================================

The following takes are illustrative of how filters can be used with conditionals::

  tasks:
    - shell: /usr/bin/foo
      register: resutl
      ignore_errors: true
    - debug: msg="previous failed"
      when: result|failed
    - debug: msg="previous changed"
      when: result|changed
    - debug: msg="previous succeeded"
      when: result|success
    - debug: msg="previous skipped"
      when: result|skipped

Forcing Variables To Be Defined
=================================

The default behavior from ansible and ansible.cfg is to fail if varialbes are undefined, but you can turn this off.

This allows an explicit check with this feature off::

  {{ variable | mandatory }}

The variable value will be used as is, but the template evaluation will raise an error if it is undefined.

Defaulting Undefined Variables
================================

Jinja2 provides a usefule *default* filter, that is often a better approach to failing if a variable is not defined::

  {{ some_variable|default(5)}}

Omitting Undefined Variables and Parameters
=============================================

It is possible to use the default filter to omit variables and module parameters using the special *omit* variable::

  - name: touch files with an optional mode
    file: dest={{item.path}} state=touch mode={{item.mode|default(omit)}}
    with_items:
      - path: /tmp/foo
      - path: /tmp/bar
        mode: "0444"

Only the final file will receive the mode=0444 option.

List Filters
==============

These filters all operate on list variables.

To get the minimum value from list of numbers::

  {{list|min}}

To get the maximum value from list of numbers::

  {{list|max}}

Set Theory Filters
====================

All these functions return a unique set from sets or lists.

To get a unique set from a list::

  {{ list1|unique }}

To get a union of two lists::

  {{ list1|union(list2) }}

To get the intersection of 2 lists::

  {{ list1|intersect(list2) }}

To get the difference of 2 lists::

  {{ list1|difference(list2) }}

To get the symmetric difference of 2 lists::

  {{ list1|symmetric_difference(list2) }}

Version Comparison Filters
============================

``version_compare`` ::

  {{ ansible_distribution_version|version_campare('12.04','>='}})

``version_compare`` filter accepts the following operators::

  <, lt, <=, le, >, gt, >=, ge, ==, =, eq, !=, <>, ne

This filter also accepts a 3rd parameter, ``strict`` which defines if strict version parsing should be used. The default is ``False``, and if set as ``True`` will use more strict version parsing::

  {{ sample_version_ver|version_compare('1.0',operator='lt',strict=True) }}

Random Number Filter
=======================

To get a random item from a list::

  {{ ['a','b','c']|random}}

To get a random number from 0 to supplied end::

  {{ 59|random }}

To get a random number from 0 to supplied end but in steps of 10::

  {{ 100|random(step=10) }}

To get a random number from 30 to supplied end but in steps of 10::

  {{ 100|random(start=30,step=10) }}

Shuffle Filter
===============

This filter will randomize an existing list, giving a different order every invocation.

To get a random list from an existing list::

  {{ ['a','b','c']|shuffle }}

Math
======

To see if something is actually a number::

  {{ myvar|isnan }}

Get the logarithm (default is e)::

  {{ myvar|log }}

Get the base 10 logarithm::

  {{ myvar|log(10) }}

Power of 2::

  {{ myvar|pow(2) }}

square root or the 5th::

  {{ myvar|root(3) }}

IP Address Filter
==================

To test if a string is a valid IP address::

  {{ myvar|ipaddr }}
  {{ myvar|ipv4 }}
  {{ myvar|ipv6 }}

Get IP address from a CIDR::

  {{ '192.0.2.1/24'|ipaddr('address') }}

Hashing filters
=================

To get the sha1 hash of a string::

  {{ 'test'|hash('sha1') }}

To get the md5 hash of a string::

  {{ 'test'|hash('md5') }}

Get a string checksum::

  {{ 'test'|checksum }}

To get a sha512 password hash (random salt)::

  {{ 'passwordsaresecret'|password_hash('sha512') }}

To get a sha256 password hash with a specific salt::

  {{ 'secretpassword'|password_hash('sha256', 'mysecretsalt') }}

Hash types available depend on the master system running ansible, ‘hash’ depends on hashlib password_hash depends on crypt.

Other Useful Filters
========================

To use one value on true and another on false::

  {{ (name == 'jhon')|ternary('Mr','Ms') }}

To concatenate a list into a string::

  {{ list|join(" ") }}

To get the last name of a file path::

  {{ path|basename }}

To get the directory name from a path::

  {{ path|dirname }}

To expand a path containing a tilde (~) character::

  {{ path|expanduser }}

To work with Base64 encoded strings::

 {{ encoded|b64decode }}
 {{ decoded|b64encode }}

To create a UUID from a string::

  {{ hostname|to_uuid }}

To match strings against a regex, use the "match" or "search" filter. ``match`` will require a complete match in the string, while ``search`` will require a match inside of the string.

::

  vars:
    url: "http://example.com/users/foo/resources/bar"
  tasks:
    - debug: "msg='matched pattern 1'"
      when: url | match("http://example.com/users/.*/resources/.*")
    - debug: "msg='matched pattern 2'"
      when: url | search("/users/.*/resources/.*")

 To replace text in a string with regex, use the ``regex_replace`` filter:: 

   # convert ansible to able
   {{ 'ansible'|regex_replace('^a.*i(.*)$','a\\1') }}

   # convert foobar to bar
   {{ 'foobar'|regex_replace('^f.*o(.*)$','\\1) }}

.. note::

  If “regex_replace” filter is used with variables inside YAML arguments (as opposed to simpler ‘key=value’ arguments), then you need to escape backreferences (e.g. \\1) with 4 backslashes (\\\\) instead of 2 (\\).
