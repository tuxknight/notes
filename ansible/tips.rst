.. _tips:

======================
Tips and Tricks
======================

.. contents:: Topics

Reboot a server and wait for it to come back
================================================

::

  - name: restart machine
    command: shutdown -r now "Ansible updates triggered"
    async: 0
    poll: 0
    ignore_errors: true

This task uses the command module to send the shutdown command to the host. By using async=0 and poll=0 , we background the process. The ``ignore_errors`` is there just to ensure that the task runs even though it is running a command that could prematurely terminate the process.

::

  - name: waiting for server to come back
    local_action: wait_for host={{inventory_hostname}}
          state=started
          delay=30
          timeout=600
          connect_timeout=30
    sudo:false

The second task waits for the server to come back online. This is a local action that was delegated to run on the Ansible control node.

::

  handlers:
    - name: restart server
      command: shutdown -r now 'Reboot triggered by Ansible'
      async: 0
      poll: 0
      ignore_errors: true

    - name: wait for server to restart
      local_action:
        module: wait_for
          host={{inventory_hostname}}
          delay=3
          timeout=600
          state=started
        sudo: false


  tasks:
    - name: Set hostname
      hostname: name=host01
      notify:
        - restart server
        - wait for server to restart

How do I split an action into a multi-line format
==================================================

To split a long task line into multiple lines, such as "action: copy src=httpd.conf dest=/etc/httpd/httpd.conf", you could format it as follows (note indentations.)::

  - name: Update the Apache config
    copy:
      src: httpd.conf
      dest: /etc/httpd/httpd.conf

Or, conversely, using the old 'action' syntax::

  - name: Update the Apache config
    action:
      module: copy
      src: httpd.conf
      dest: /etc/httpd/httpd.conf

Use YAML line continuations::

  - name: Update the Apache config
    copy:>
      src=httpd.conf
      dest=/etc/httpd/httpd.conf

Or::

  - name: Update the Apache config
    action: copy >
      src=httpd.conf
      dest=/etc/httpd/httpd.conf
