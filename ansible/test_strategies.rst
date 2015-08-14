.. _test_strategies:

=================
Test Strategies
=================

.. contents:: Topics

Check Mode as A Drift Test
============================

``--check`` mode is Ansible can be used as a layer of testing as well. Using the ``--check`` flag to the ansible command will report if Ansible thinks it would have had to have make any changes to bring the system into a desired state.

Ordinarily scripts and commands dont run in check mode, so if you want certain steps to always execute in check mode, add the ``always_run`` flag::

  tasks:
    - script: varify.sh
      always_run: True

Modules That Are Useful for Testing
======================================

Certain playbook modules are particularly good for testing::

  task:
    - wait_for: host={{inventory_hostname}} port=22
      delegate_to: localhost
      register: wait
      ignore_errors: true

    - fail: msg="port start failed."
      when: wait.failed

>WAIT_FOR

  Waiting for a port to become available is useful for when services
  are not immediately available after their init scripts return -
  which is true of certain Java application servers. It is also
  useful when starting guests with the [virt] module and needing to
  pause until they are ready. This module can also be used to wait
  for a file to be available on the filesystem or with a regex match
  a string to be present in a file.

::

  tasks:
    - uri: url=http://www.baidu.com return_content=yes
      register: webpage

    - fail: msg='service is not start'
      when: "'something' not in webpage.content"

It is easy to push an arbitrary script on a remote host and the script will automatically fail if it has a non-zero return code::

  tasks:
    - script: test_script
      registry: r_code

    - debug: msg="{{r_code}}"

With the assert module makes it very easy to validate various kinds of truth::

  tasks:
    - shell: /usr/bin/some-command
      register: cmd_result

    - assert:
        that:
          - "'not ready' not in cmd_result.stderr"
          - "'gizmo enabled' in cmd_result.stdout"

>ASSERT

 This module asserts that a given expression is true and can be a
 simpler alternative to the 'fail' module in some cases.

You could use the ``stat`` module to test for existence of files that are not declaratively set by your Ansible configuration::

  tasks:
    - stat: path=/path/to/test/
      register: f
    - assert:
        that:
          - f.stat.exists and f.stat.isdir

>STAT

  Retrieves facts for a file similar to the linux/unix 'stat'
  command.

There is no need to check things like the return code of commands, Ansible is checking them automatically. Rather than checking for a user to exist, consider using the user module to make it exist.


Integrating Testing With Rolling Updates
===========================================

This is the great culmination of embedded tests::

  ---
  - hosts: webservers
    serial: 5
    pre_tasks:
      - name: take out of load balancer pool
        command: /usr/bin/task_out_of_pool
        delegate_to: 127.0.0.1

    roles:
      - common
      - webserver
      - apply_testing_checks
    or
    tasks:
      - script: /srv/qa_team/app_testing_script.sh
        delegate_to: testing_server

    post_tasks:
      - name: add back
        command: /usr/bin/add_back_to_pool
        delegate_to: 127.0.0.1

If the "apply_testing_checks" step is not performed the machine will not go back into the pool.

Achieving Continuous Deployment
=================================

The workflow may look like this::

  - Write and use antomation to deploy local development VMs
  - Have a CI system like jenkins deploy to a stage environment on every code change
  - The deploy job calls testing scripts to pass/fail a build on every deploy
  - If the deploy job succeeds, it runs the same deploy playbook against production inventory

Conclusion
============

Ansible believes you should not need another framework to validate basic things of your infrastructure is true. Ansible is an order-based system that will fail immediately on unhandled errors for a host, and prevent further configuration of that host.

Since Ansible is designed as a multi-tier orchestration system, it makes it very easy to incorporate tests into the end of a playbook run, either using loose tasks or roles.

Finally, because Ansible errors propagate all the way up to the return code of the Ansible program itself, and Ansible by default runs in an easy push-based mode, Ansible is a great step to put into a build environment if you wish to use it to roll out systems as part of a Continuous Integration/Continuous Delivery pipeline, as is covered in sections above.

The focus should not be on infrastructure testing, but on application testing. Obviously at the development stage, unit tests are great too.Ansible describes states of resources declaratively, so you dont have to do unit test on your playbook.

In all, testing is a very organizational and site-specific thing. Everybody should be doing it, but what makes the most sense for your environment will vary with what you are deploying and who is using it – but everyone benefits from a more robust and reliable deployment system.

