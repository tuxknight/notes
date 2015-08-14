.. _ansible_module_doc:

====================
Ansible Module Docs
====================


> ACL
=======================

::

    Sets and retrieves file ACL information.

    Options (= is mandatory):

    - default
          if the target is a directory, setting this to yes will make
          it the default acl for entities created inside the
          directory. It causes an error if name is a file. (Choices:
          yes, no)

    - entity
          actual user or group that the ACL applies to when matching
          entity types user or group are selected.

    - entry
          DEPRECATED. The acl to set or remove.  This must always be
          quoted in the form of '<etype>:<qualifier>:<perms>'.  The
          qualifier may be empty for some types, but the type and
          perms are always requried. '-' can be used as placeholder
          when you do not care about permissions. This is now
          superceeded by entity, type and permissions fields.

    - etype
          if the target is a directory, setting this to yes will make
          it the default acl for entities created inside the
          directory. It causes an error if name is a file. (Choices:
          user, group, mask, other)

    - follow
          whether to follow symlinks on the path if a symlink is
          encountered. (Choices: yes, no)

    = name
          The full path of the file or object.

    - permissions
          Permissions to apply/remove can be any combination of r, w
          and  x (read, write and execute respectively)

    - state
          defines whether the ACL should be present or not.  The
          `query' state gets the current acl `present' without
          changing it, for use in 'register' operations. (Choices:
          query, present, absent)

    Notes:    The "acl" module requires that acls are enabled on the target
          filesystem and that the setfacl and getfacl binaries are
          installed.

    # Grant user Joe read access to a file
    - acl: name=/etc/foo.conf entity=joe etype=user permissions="r" state=present
    
    # Removes the acl for Joe on a specific file
    - acl: name=/etc/foo.conf entity=joe etype=user state=absent
    
    # Sets default acl for joe on foo.d
    - acl: name=/etc/foo.d entity=joe etype=user permissions=rw default=yes state=present
    
    # Same as previous but using entry shorthand
    - acl: name=/etc/foo.d entrty="default:user:joe:rw-" state=present
    
    # Obtain the acl for a specific file
    - acl: name=/etc/foo.conf
      register: acl_info

> ADD_HOST
=======================

::

    Use variables to create new hosts and groups in inventory for use
    in later plays of the same playbook. Takes variables so you can
    define the new hosts more fully.

    Options (= is mandatory):

    - groups
          The groups to add the hostname to, comma separated.

    = name
          The hostname/ip of the host to add to the inventory, can
          include a colon and a port number.

    # add host to group 'just_created' with variable foo=42
    - add_host: name={{ ip_from_ec2 }} groups=just_created foo=42
    
    # add a host with a non-standard port local to your machines
    - add_host: name={{ new_ip }}:{{ new_port }}
    
    # add a host alias that we reach through a tunnel
    - add_host: hostname={{ new_ip }}
                ansible_ssh_host={{ inventory_hostname }}
                ansible_ssh_port={{ new_port }}

> AIRBRAKE_DEPLOYMENT
=======================

::

    Notify airbrake about app deployments (see
    http://help.airbrake.io/kb/api-2/deploy-tracking)

    Options (= is mandatory):

    = environment
          The airbrake environment name, typically 'production',
          'staging', etc.

    - repo
          URL of the project repository

    - revision
          A hash, number, tag, or other identifier showing what
          revision was deployed

    = token
          API token.

    - url
          Optional URL to submit the notification to. Use to send
          notifications to Airbrake-compliant tools like Errbit.

    - user
          The username of the person doing the deployment

    - validate_certs
          If `no', SSL certificates for the target url will not be
          validated. This should only be used on personally controlled
          sites using self-signed certificates. (Choices: yes, no)

    Requirements:    urllib, urllib2

    - airbrake_deployment: token=AAAAAA
                           environment='staging'
                           user='ansible'
                           revision=4.2

> APT
=======================

::

    Manages `apt' packages (such as for Debian/Ubuntu).

    Options (= is mandatory):

    - cache_valid_time
          If `update_cache' is specified and the last run is less or
          equal than `cache_valid_time' seconds ago, the
          `update_cache' gets skipped.

    - default_release
          Corresponds to the `-t' option for `apt' and sets pin
          priorities

    - dpkg_options
          Add dpkg options to apt command. Defaults to '-o
          "Dpkg::Options::=--force-confdef" -o "Dpkg::Options
          ::=--force-confold"'Options should be supplied as comma
          separated list

    - force
          If `yes', force installs/removes. (Choices: yes, no)

    - install_recommends
          Corresponds to the `--no-install-recommends' option for
          `apt', default behavior works as apt's default behavior,
          `no' does not install recommended packages. Suggested
          packages are never installed. (Choices: yes, no)

    - pkg
          A package name or package specifier with version, like `foo'
          or `foo=1.0'. Shell like wildcards (fnmatch) like apt* are
          also supported.

    - purge
          Will force purging of configuration files if the module
          state is set to `absent'. (Choices: yes, no)

    - state
          Indicates the desired package state (Choices: latest,
          absent, present)

    - update_cache
          Run the equivalent of `apt-get update' before the operation.
          Can be run as part of the package installation or as a
          separate step (Choices: yes, no)

    - upgrade
          If yes or safe, performs an aptitude safe-upgrade.If full,
          performs an aptitude full-upgrade.If dist, performs an apt-
          get dist-upgrade.Note: This does not upgrade a specific
          package, use state=latest for that. (Choices: yes, safe,
          full, dist)

    Notes:    Three of the upgrade modes (`full', `safe' and its alias `yes')
          require `aptitude', otherwise `apt-get' suffices.

    Requirements:    python-apt, aptitude

    # Update repositories cache and install "foo" package
    - apt: pkg=foo update_cache=yes
    
    # Remove "foo" package
    - apt: pkg=foo state=absent
    
    # Install the package "foo"
    - apt: pkg=foo state=present
    
    # Install the version '1.00' of package "foo"
    - apt: pkg=foo=1.00 state=present
    
    # Update the repository cache and update package "nginx" to latest version using default release squeeze-backport
    - apt: pkg=nginx state=latest default_release=squeeze-backports update_cache=yes
    
    # Install latest version of "openjdk-6-jdk" ignoring "install-recommends"
    - apt: pkg=openjdk-6-jdk state=latest install_recommends=no
    
    # Update all packages to the latest version
    - apt: upgrade=dist
    
    # Run the equivalent of "apt-get update" as a separate step
    - apt: update_cache=yes
    
    # Only run "update_cache=yes" if the last one is more than more than 3600 seconds ago
    - apt: update_cache=yes cache_valid_time=3600
    
    # Pass options to dpkg on run
    - apt: upgrade=dist update_cache=yes dpkg_options='force-confold,force-confdef'

> APT_KEY
=======================

::

    Add or remove an `apt' key, optionally downloading it

    Options (= is mandatory):

    - data
          keyfile contents

    - file
          keyfile path

    - id
          identifier of key

    - keyring
          path to specific keyring file in /etc/apt/trusted.gpg.d

    - state
          used to specify if key is being added or revoked (Choices:
          absent, present)

    - url
          url to retrieve key from.

    - validate_certs
          If `no', SSL certificates for the target url will not be
          validated. This should only be used on personally controlled
          sites using self-signed certificates. (Choices: yes, no)

    Notes:    doesn't download the key unless it really needs itas a sanity
          check, downloaded key id must match the one specifiedbest
          practice is to specify the key id and the url

    # Add an Apt signing key, uses whichever key is at the URL
    - apt_key: url=https://ftp-master.debian.org/keys/archive-key-6.0.asc state=present
    
    # Add an Apt signing key, will not download if present
    - apt_key: id=473041FA url=https://ftp-master.debian.org/keys/archive-key-6.0.asc state=present
    
    # Remove an Apt signing key, uses whichever key is at the URL
    - apt_key: url=https://ftp-master.debian.org/keys/archive-key-6.0.asc state=absent
    
    # Remove a Apt specific signing key, leading 0x is valid
    - apt_key: id=0x473041FA state=absent
    
    # Add a key from a file on the Ansible server
    - apt_key: data="{{ lookup('file', 'apt.gpg') }}" state=present
    
    # Add an Apt signing key to a specific keyring file
    - apt_key: id=473041FA url=https://ftp-master.debian.org/keys/archive-key-6.0.asc keyring=/etc/apt/trusted.gpg.d/debian.gpg state=present

> APT_REPOSITORY
=======================

::

    Add or remove an APT repositories in Ubuntu and Debian.

    Options (= is mandatory):

    = repo
          A source string for the repository.

    - state
          A source string state. (Choices: absent, present)

    - update_cache
          Run the equivalent of `apt-get update' if has changed.
          (Choices: yes, no)

    Notes:    This module works on Debian and Ubuntu and requires `python-apt'
          and `python-pycurl' packages.This module supports Debian
          Squeeze (version 6) as well as its successors.This module
          treats Debian and Ubuntu distributions separately. So PPA
          could be installed only on Ubuntu machines.

    Requirements:    python-apt, python-pycurl

    # Add specified repository into sources list.
    apt_repository: repo='deb http://archive.canonical.com/ubuntu hardy partner' state=present
    
    # Add source repository into sources list.
    apt_repository: repo='deb-src http://archive.canonical.com/ubuntu hardy partner' state=present
    
    # Remove specified repository from sources list.
    apt_repository: repo='deb http://archive.canonical.com/ubuntu hardy partner' state=absent
    
    # On Ubuntu target: add nginx stable repository from PPA and install its signing key.
    # On Debian target: adding PPA is not available, so it will fail immediately.
    apt_repository: repo='ppa:nginx/stable'

> ARISTA_INTERFACE
=======================

::

    Manage physical Ethernet interface resources on Arista EOS network
    devices

    Options (= is mandatory):

    - admin
          controls the operational state of the interface (Choices:
          up, down)

    - description
          a single line text string describing the interface

    - duplex
          sets the interface duplex setting (Choices: auto, half,
          full)

    = interface_id
          the full name of the interface

    - logging
          enables or disables the syslog facility for this module
          (Choices: true, false, yes, no)

    - mtu
          configureds the maximum transmission unit for the interface

    - speed
          sets the interface speed setting (Choices: auto, 100m, 1g,
          10g)

    Notes:    Requires EOS 4.10 or laterThe Netdev extension for EOS must be
          installed and active in the available extensions (show
          extensions from the EOS CLI)See
          http://eos.aristanetworks.com for details

    Requirements:    Arista EOS 4.10, Netdev extension for EOS

    Example playbook entries using the arista_interface module to manage resource 
    state.  Note that interface names must be the full interface name not shortcut
    names (ie Ethernet, not Et1)
    
        tasks:
        - name: enable interface Ethernet 1
          action: arista_interface interface_id=Ethernet1 admin=up speed=10g duplex=full logging=true
      
        - name: set mtu on Ethernet 1
          action: arista_interface interface_id=Ethernet1 mtu=1600 speed=10g duplex=full logging=true
      
        - name: reset changes to Ethernet 1
          action: arista_interface interface_id=Ethernet1 admin=down mtu=1500 speed=10g duplex=full logging=true

> ARISTA_L2INTERFACE
=======================

::

    Manage layer 2 interface resources on Arista EOS network devices

    Options (= is mandatory):

    = interface_id
          the full name of the interface

    - logging
          enables or disables the syslog facility for this module
          (Choices: true, false, yes, no)

    - state
          describe the desired state of the interface related to the
          config (Choices: present, absent)

    - tagged_vlans
          specifies the list of vlans that should be allowed to
          transit this interface

    - untagged_vlan
          specifies the vlan that untagged traffic should be placed in
          for transit across a vlan tagged link

    - vlan_tagging
          specifies whether or not vlan tagging should be enabled for
          this interface (Choices: enable, disable)

    Notes:    Requires EOS 4.10 or laterThe Netdev extension for EOS must be
          installed and active in the available extensions (show
          extensions from the EOS CLI)See
          http://eos.aristanetworks.com for details

    Requirements:    Arista EOS 4.10, Netdev extension for EOS

    Example playbook entries using the arista_l2interface module to manage resource 
    state. Note that interface names must be the full interface name not shortcut
    names (ie Ethernet, not Et1)
    
        tasks:
        - name: create switchport ethernet1 access port 
          action: arista_l2interface interface_id=Ethernet1 logging=true
    
        - name: create switchport ethernet2 trunk port
          action: arista_l2interface interface_id=Ethernet2 vlan_tagging=enable logging=true
    
        - name: add vlans to red and blue switchport ethernet2
          action: arista_l2interface interface_id=Ethernet2 tagged_vlans=red,blue logging=true
    
        - name: set untagged vlan for Ethernet1
          action: arista_l2interface interface_id=Ethernet1 untagged_vlan=red logging=true
    
        - name: convert access to trunk
          action: arista_l2interface interface_id=Ethernet1 vlan_tagging=enable tagged_vlans=red,blue logging=true
    
        - name: convert trunk to access
          action: arista_l2interface interface_id=Ethernet2 vlan_tagging=disable untagged_vlan=blue logging=true
    
        - name: delete switchport ethernet1
          action: arista_l2interface interface_id=Ethernet1 state=absent logging=true

> ARISTA_LAG
=======================

::

    Manage port channel interface resources on Arista EOS network
    devices

    Options (= is mandatory):

    = interface_id
          the full name of the interface

    - lacp
          enables the use of the LACP protocol for managing link
          bundles (Choices: active, passive, off)

    - links
          array of physical interface links to include in this lag

    - logging
          enables or disables the syslog facility for this module
          (Choices: true, false, yes, no)

    - minimum_links
          the minimum number of physical interaces that must be
          operationally up to consider the lag operationally up

    - state
          describe the desired state of the interface related to the
          config (Choices: present, absent)

    Notes:    Requires EOS 4.10 or laterThe Netdev extension for EOS must be
          installed and active in the available extensions (show
          extensions from the EOS CLI)See
          http://eos.aristanetworks.com for details

    Requirements:    Arista EOS 4.10, Netdev extension for EOS

    Example playbook entries using the arista_lag module to manage resource 
    state.  Note that interface names must be the full interface name not shortcut
    names (ie Ethernet, not Et1)
    
        tasks:
        - name: create lag interface
          action: arista_lag interface_id=Port-Channel1 links=Ethernet1,Ethernet2 logging=true
    
        - name: add member links
          action: arista_lag interface_id=Port-Channel1 links=Ethernet1,Ethernet2,Ethernet3 logging=true
    
        - name: remove member links
          action: arista_lag interface_id=Port-Channel1 links=Ethernet2,Ethernet3 logging=true
        
        - name: remove lag interface
          action: arista_lag interface_id=Port-Channel1 state=absent logging=true

> ARISTA_VLAN
=======================

::

    Manage VLAN resources on Arista EOS network devices.  This module
    requires the Netdev EOS extension to be installed in EOS.  For
    detailed instructions for installing and using the Netdev module
    please see [link]

    Options (= is mandatory):

    - logging
          enables or disables the syslog facility for this module
          (Choices: true, false, yes, no)

    - name
          a descriptive name for the vlan

    - state
          describe the desired state of the vlan related to the config
          (Choices: present, absent)

    = vlan_id
          the vlan id

    Notes:    Requires EOS 4.10 or laterThe Netdev extension for EOS must be
          installed and active in the available extensions (show
          extensions from the EOS CLI)See
          http://eos.aristanetworks.com for details

    Requirements:    Arista EOS 4.10, Netdev extension for EOS

    Example playbook entries using the arista_vlan module to manage resource 
    state.  
       
      tasks:
      - name: create vlan 999
        action: arista_vlan vlan_id=999 logging=true
        
      - name: create / edit vlan 999
        action: arista_vlan vlan_id=999 name=test logging=true
        
      - name: remove vlan 999
        action: arista_vlan vlan_id=999 state=absent logging=true
        

> ASSEMBLE
=======================

::

    Assembles a configuration file from fragments. Often a particular
    program will take a single configuration file and does not support
    a `conf.d' style structure where it is easy to build up the
    configuration from multiple sources. [assemble] will take a
    directory of files that can be local or have already been
    transferred to the system, and concatenate them together to
    produce a destination file. Files are assembled in string sorting
    order. Puppet calls this idea `fragments'.

    Options (= is mandatory):

    - backup
          Create a backup file (if `yes'), including the timestamp
          information so you can get the original file back if you
          somehow clobbered it incorrectly. (Choices: yes, no)

    - delimiter
          A delimiter to seperate the file contents.

    = dest
          A file to create using the concatenation of all of the
          source files.

    - others
          all arguments accepted by the [file] module also work here

    - regexp
          Assemble files only if `regex' matches the filename. If not
          set, all files are assembled. All "" (backslash) must be
          escaped as "\\" to comply yaml syntax. Uses Python regular
          expressions; see http://docs.python.org/2/library/re.html.

    - remote_src
          If False, it will search for src at originating/master
          machine, if True it will go to the remote/target machine for
          the src. Default is True. (Choices: True, False)

    = src
          An already existing directory full of source files.

    # Example from Ansible Playbooks
    - assemble: src=/etc/someapp/fragments dest=/etc/someapp/someapp.conf
    
    # When a delimiter is specified, it will be inserted in between each fragment
    - assemble: src=/etc/someapp/fragments dest=/etc/someapp/someapp.conf delimiter='### START FRAGMENT ###'

> ASSERT
=======================

::

    This module asserts that a given expression is true and can be a
    simpler alternative to the 'fail' module in some cases.

    Options (= is mandatory):

    = that
          A string expression of the same form that can be passed to
          the 'when' statement

    - assert: ansible_os_family != "RedHat"
    - assert: "'foo' in some_command_result.stdout"

> AT
=======================

::

    Use this module to schedule a command or script to run once in the
    future.All jobs are executed in the a queue.

    Options (= is mandatory):

    = action
          The action to take for the job defaulting to add. Unique
          will verify that there is only one entry in the queue.Delete
          will remove all existing queued jobs. (Choices: add, delete,
          unique)

    - command
          A command to be executed in the future.

    - script_file
          An existing script to be executed in the future.

    = unit_count
          The count of units in the future to execute the command or
          script.

    = unit_type
          The type of units in the future to execute the command or
          script. (Choices: minutes, hours, days, weeks)

    - user
          The user to execute the at command as.

    Requirements:    at

    # Schedule a command to execute in 20 minutes as root.
    - at: command="ls -d / > /dev/null" unit_count=20 unit_type="minutes"
    
    # Schedule a script to execute in 1 hour as the neo user.
    - at: script_file="/some/script.sh" user="neo" unit_count=1 unit_type="hours"
    
    # Match a command to an existing job and delete the job.
    - at: command="ls -d / > /dev/null" action="delete"
    
    # Schedule a command to execute in 20 minutes making sure it is unique in the queue.
    - at: command="ls -d / > /dev/null" action="unique" unit_count=20 unit_type="minutes"

> AUTHORIZED_KEY
=======================

::

    Adds or removes authorized keys for particular user accounts

    Options (= is mandatory):

    = key
          The SSH public key, as a string

    - key_options
          A string of ssh key options to be prepended to the key in
          the authorized_keys file

    - manage_dir
          Whether this module should manage the directory of the
          authorized_keys file. Make sure to set `manage_dir=no' if
          you are using an alternate directory for authorized_keys set
          with `path', since you could lock yourself out of SSH
          access. See the example below. (Choices: yes, no)

    - path
          Alternate path to the authorized_keys file

    - state
          Whether the given key (with the given key_options) should or
          should not be in the file (Choices: present, absent)

    = user
          The username on the remote host whose authorized_keys file
          will be modified

    # Example using key data from a local file on the management machine
    - authorized_key: user=charlie key="{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
    
    # Using alternate directory locations:
    - authorized_key: user=charlie
                      key="{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
                      path='/etc/ssh/authorized_keys/charlie'
                      manage_dir=no
    
    # Using with_file
    - name: Set up authorized_keys for the deploy user
      authorized_key: user=deploy
                      key="{{ item }}"
      with_file:
        - public_keys/doe-jane
        - public_keys/doe-john
    
    # Using key_options:
    - authorized_key: user=charlie
                      key="{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
                      key_options='no-port-forwarding,host="10.0.1.1"'

> BIGIP_MONITOR_HTTP
=======================

::

    Manages F5 BIG-IP LTM monitors via iControl SOAP API

    Options (= is mandatory):

    - interval
          The interval specifying how frequently the monitor instance
          of this template will run. By default, this interval is used
          for up and down states. The default API setting is 5.

    - ip
          IP address part of the ipport definition. The default API
          setting is "0.0.0.0".

    = name
          Monitor name

    - parent
          The parent template of this monitor template

    - parent_partition
          Partition for the parent monitor

    - partition
          Partition for the monitor

    = password
          BIG-IP password

    - port
          port address part op the ipport definition. Tyhe default API
          setting is 0.

    = receive
          The receive string for the monitor call

    = receive_disable
          The receive disable string for the monitor call

    = send
          The send string for the monitor call

    = server
          BIG-IP host

    - state
          Monitor state (Choices: present, absent)

    - time_until_up
          Specifies the amount of time in seconds after the first
          successful response before a node will be marked up. A value
          of 0 will cause a node to be marked up immediately after a
          valid response is received from the node. The default API
          setting is 0.

    - timeout
          The number of seconds in which the node or service must
          respond to the monitor request. If the target responds
          within the set time period, it is considered up. If the
          target does not respond within the set time period, it is
          considered down. You can change this number to any number
          you want, however, it should be 3 times the interval number
          of seconds plus 1 second. The default API setting is 16.

    = user
          BIG-IP username

    Notes:    Requires BIG-IP software version >= 11F5 developed module
          'bigsuds' required (see http://devcentral.f5.com)Best run as
          a local_action in your playbookMonitor API documentation: ht
          tps://devcentral.f5.com/wiki/iControl.LocalLB__Monitor.ashx

    Requirements:    bigsuds

    - name: BIGIP F5 | Create HTTP Monitor
      local_action:
        module:             bigip_monitor_http
        state:              present
        server:             "{{ f5server }}"
        user:               "{{ f5user }}"
        password:           "{{ f5password }}"
        name:               "{{ item.monitorname }}"
        send:               "{{ item.send }}"
        receive:            "{{ item.receive }}"
      with_items: f5monitors
    - name: BIGIP F5 | Remove HTTP Monitor
      local_action:
        module:             bigip_monitor_http
        state:              absent
        server:             "{{ f5server }}"
        user:               "{{ f5user }}"
        password:           "{{ f5password }}"
        name:               "{{ monitorname }}"

> BIGIP_MONITOR_TCP
=======================

::

    Manages F5 BIG-IP LTM tcp monitors via iControl SOAP API

    Options (= is mandatory):

    - interval
          The interval specifying how frequently the monitor instance
          of this template will run. By default, this interval is used
          for up and down states. The default API setting is 5.

    - ip
          IP address part of the ipport definition. The default API
          setting is "0.0.0.0".

    = name
          Monitor name

    - parent
          The parent template of this monitor template (Choices: tcp,
          tcp_echo, tcp_half_open)

    - parent_partition
          Partition for the parent monitor

    - partition
          Partition for the monitor

    = password
          BIG-IP password

    - port
          port address part op the ipport definition. Tyhe default API
          setting is 0.

    = receive
          The receive string for the monitor call

    = send
          The send string for the monitor call

    = server
          BIG-IP host

    - state
          Monitor state (Choices: present, absent)

    - time_until_up
          Specifies the amount of time in seconds after the first
          successful response before a node will be marked up. A value
          of 0 will cause a node to be marked up immediately after a
          valid response is received from the node. The default API
          setting is 0.

    - timeout
          The number of seconds in which the node or service must
          respond to the monitor request. If the target responds
          within the set time period, it is considered up. If the
          target does not respond within the set time period, it is
          considered down. You can change this number to any number
          you want, however, it should be 3 times the interval number
          of seconds plus 1 second. The default API setting is 16.

    - type
          The template type of this monitor template (Choices:
          TTYPE_TCP, TTYPE_TCP_ECHO, TTYPE_TCP_HALF_OPEN)

    = user
          BIG-IP username

    Notes:    Requires BIG-IP software version >= 11F5 developed module
          'bigsuds' required (see http://devcentral.f5.com)Best run as
          a local_action in your playbookMonitor API documentation: ht
          tps://devcentral.f5.com/wiki/iControl.LocalLB__Monitor.ashx

    Requirements:    bigsuds

    
    - name: BIGIP F5 | Create TCP Monitor
      local_action:
        module:             bigip_monitor_tcp
        state:              present
        server:             "{{ f5server }}"
        user:               "{{ f5user }}"
        password:           "{{ f5password }}"
        name:               "{{ item.monitorname }}"
        type:               tcp
        send:               "{{ item.send }}"
        receive:            "{{ item.receive }}"
      with_items: f5monitors-tcp
    - name: BIGIP F5 | Create TCP half open Monitor
      local_action:
        module:             bigip_monitor_tcp
        state:              present
        server:             "{{ f5server }}"
        user:               "{{ f5user }}"
        password:           "{{ f5password }}"
        name:               "{{ item.monitorname }}"
        type:               tcp
        send:               "{{ item.send }}"
        receive:            "{{ item.receive }}"
      with_items: f5monitors-halftcp
    - name: BIGIP F5 | Remove TCP Monitor
      local_action:
        module:             bigip_monitor_tcp
        state:              absent
        server:             "{{ f5server }}"
        user:               "{{ f5user }}"
        password:           "{{ f5password }}"
        name:               "{{ monitorname }}"
      with_flattened:
      - f5monitors-tcp
      - f5monitors-halftcp
    

> BIGIP_NODE
=======================

::

    Manages F5 BIG-IP LTM nodes via iControl SOAP API

    Options (= is mandatory):

    - description
          Node description. (Choices: )

    = host
          Node IP. Required when state=present and node does not
          exist. Error when state=absent. (Choices: )

    - name
          Node name (Choices: )

    - partition
          Partition (Choices: )

    = password
          BIG-IP password (Choices: )

    = server
          BIG-IP host (Choices: )

    = state
          Pool member state (Choices: present, absent)

    = user
          BIG-IP username (Choices: )

    Notes:    Requires BIG-IP software version >= 11F5 developed module
          'bigsuds' required (see http://devcentral.f5.com)Best run as
          a local_action in your playbook

    Requirements:    bigsuds

    
    ## playbook task examples:
    
    ---
    # file bigip-test.yml
    # ...
    - hosts: bigip-test
      tasks:
      - name: Add node
        local_action: >
          bigip_node
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          partition=matthite
          host="{{ ansible_default_ipv4["address"] }}"
          name="{{ ansible_default_ipv4["address"] }}"
    
    # Note that the BIG-IP automatically names the node using the
    # IP address specified in previous play's host parameter.
    # Future plays referencing this node no longer use the host
    # parameter but instead use the name parameter.
    # Alternatively, you could have specified a name with the
    # name parameter when state=present.
    
      - name: Modify node description
        local_action: >
          bigip_node
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          partition=matthite
          name="{{ ansible_default_ipv4["address"] }}"
          description="Our best server yet"
    
      - name: Delete node
        local_action: >
          bigip_node
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=absent
          partition=matthite
          name="{{ ansible_default_ipv4["address"] }}"
    

> BIGIP_POOL
=======================

::

    Manages F5 BIG-IP LTM pools via iControl SOAP API

    Options (= is mandatory):

    - host
          Pool member IP (Choices: )

    - lb_method
          Load balancing method (Choices: round_robin, ratio_member,
          least_connection_member, observed_member, predictive_member,
          ratio_node_address, least_connection_node_address,
          fastest_node_address, observed_node_address,
          predictive_node_address, dynamic_ratio,
          fastest_app_response, least_sessions, dynamic_ratio_member,
          l3_addr, unknown, weighted_least_connection_member,
          weighted_least_connection_node_address, ratio_session,
          ratio_least_connection_member,
          ratio_least_connection_node_address)

    - monitor_type
          Monitor rule type when monitors > 1 (Choices: and_list,
          m_of_n)

    - monitors
          Monitor template name list. Always use the full path to the
          monitor. (Choices: )

    = name
          Pool name (Choices: )

    - partition
          Partition of pool/pool member (Choices: )

    = password
          BIG-IP password (Choices: )

    - port
          Pool member port (Choices: )

    - quorum
          Monitor quorum value when monitor_type is m_of_n (Choices: )

    = server
          BIG-IP host (Choices: )

    - service_down_action
          Sets the action to take when node goes down in pool
          (Choices: none, reset, drop, reselect)

    - slow_ramp_time
          Sets the ramp-up time (in seconds) to gradually ramp up the
          load on newly added or freshly detected up pool members
          (Choices: )

    - state
          Pool/pool member state (Choices: present, absent)

    = user
          BIG-IP username (Choices: )

    Notes:    Requires BIG-IP software version >= 11F5 developed module
          'bigsuds' required (see http://devcentral.f5.com)Best run as
          a local_action in your playbook

    Requirements:    bigsuds

    
    ## playbook task examples:
    
    ---
    # file bigip-test.yml
    # ...
    - hosts: localhost
      tasks:
      - name: Create pool
        local_action: >
          bigip_pool
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          name=matthite-pool
          partition=matthite
          lb_method=least_connection_member
          slow_ramp_time=120
    
      - name: Modify load balancer method
        local_action: >
          bigip_pool
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          name=matthite-pool
          partition=matthite
          lb_method=round_robin
    
    - hosts: bigip-test
      tasks:
      - name: Add pool member
        local_action: >
          bigip_pool
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          name=matthite-pool
          partition=matthite
          host="{{ ansible_default_ipv4["address"] }}"
          port=80
    
      - name: Remove pool member from pool
        local_action: >
          bigip_pool
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=absent
          name=matthite-pool
          partition=matthite
          host="{{ ansible_default_ipv4["address"] }}"
          port=80
    
    - hosts: localhost
      tasks:
      - name: Delete pool
        local_action: >
          bigip_pool
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=absent
          name=matthite-pool
          partition=matthite
    

> BIGIP_POOL_MEMBER
=======================

::

    Manages F5 BIG-IP LTM pool members via iControl SOAP API

    Options (= is mandatory):

    - connection_limit
          Pool member connection limit. Setting this to 0 disables the
          limit. (Choices: )

    - description
          Pool member description (Choices: )

    = host
          Pool member IP (Choices: )

    - partition
          Partition (Choices: )

    = password
          BIG-IP password (Choices: )

    = pool
          Pool name. This pool must exist. (Choices: )

    = port
          Pool member port (Choices: )

    - rate_limit
          Pool member rate limit (connections-per-second). Setting
          this to 0 disables the limit. (Choices: )

    - ratio
          Pool member ratio weight. Valid values range from 1 through
          100. New pool members -- unless overriden with this value --
          default to 1. (Choices: )

    = server
          BIG-IP host (Choices: )

    = state
          Pool member state (Choices: present, absent)

    = user
          BIG-IP username (Choices: )

    Notes:    Requires BIG-IP software version >= 11F5 developed module
          'bigsuds' required (see http://devcentral.f5.com)Best run as
          a local_action in your playbookSupersedes bigip_pool for
          managing pool members

    Requirements:    bigsuds

    
    ## playbook task examples:
    
    ---
    # file bigip-test.yml
    # ...
    - hosts: bigip-test
      tasks:
      - name: Add pool member
        local_action: >
          bigip_pool_member
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          pool=matthite-pool
          partition=matthite
          host="{{ ansible_default_ipv4["address"] }}"
          port=80
          description="web server"
          connection_limit=100
          rate_limit=50
          ratio=2
    
      - name: Modify pool member ratio and description
        local_action: >
          bigip_pool_member
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=present
          pool=matthite-pool
          partition=matthite
          host="{{ ansible_default_ipv4["address"] }}"
          port=80
          ratio=1
          description="nginx server"
    
      - name: Remove pool member from pool
        local_action: >
          bigip_pool_member
          server=lb.mydomain.com
          user=admin
          password=mysecret
          state=absent
          pool=matthite-pool
          partition=matthite
          host="{{ ansible_default_ipv4["address"] }}"
          port=80
    

> BOUNDARY_METER
=======================

::

    This module manages boundary meters

    Options (= is mandatory):

    = apiid
          Organizations boundary API ID

    = apikey
          Organizations boundary API KEY

    = name
          meter name

    - state
          Whether to create or remove the client from boundary
          (Choices: present, absent)

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Notes:    This module does not yet support boundary tags.

    Requirements:    Boundary API access, bprobe is required to send data, but not to
          register a meter, Python urllib2

    - name: Create meter
      boundary_meter: apiid=AAAAAA api_key=BBBBBB state=present name={{ inventory_hostname }}"
    
    - name: Delete meter
      boundary_meter: apiid=AAAAAA api_key=BBBBBB state=absent name={{ inventory_hostname }}"
    

> BZR
=======================

::

    Manage `bzr' branches to deploy files or software.

    Options (= is mandatory):

    = dest
          Absolute path of where the branch should be cloned to.

    - executable
          Path to bzr executable to use. If not supplied, the normal
          mechanism for resolving binary paths will be used.

    - force
          If `yes', any modified files in the working tree will be
          discarded. (Choices: yes, no)

    = name
          SSH or HTTP protocol address of the parent branch.

    - version
          What version of the branch to clone.  This can be the bzr
          revno or revid.

    # Example bzr checkout from Ansible Playbooks
    - bzr: name=bzr+ssh://foosball.example.org/path/to/branch dest=/srv/checkout version=22

> CAMPFIRE
=======================

::

    Send a message to Campfire.Messages with newlines will result in a
    "Paste" message being sent.

    Options (= is mandatory):

    = msg
          The message body.

    - notify
          Send a notification sound before the message. (Choices: 56k,
          bueller, crickets, dangerzone, deeper, drama, greatjob,
          horn, horror, inconceivable, live, loggins, noooo, nyan,
          ohmy, ohyeah, pushit, rimshot, sax, secret, tada, tmyk,
          trombone, vuvuzela, yeah, yodel)

    = room
          Room number to which the message should be sent.

    = subscription
          The subscription name to use.

    = token
          API token.

    Requirements:    urllib2, cgi

    - campfire: subscription=foo token=12345 room=123 msg="Task completed."
    
    - campfire: subscription=foo token=12345 room=123 notify=loggins
            msg="Task completed ... with feeling."

> CLOUDFORMATION
=======================

::

    Launches an AWS CloudFormation stack and waits for it complete.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - disable_rollback
          If a stacks fails to form, rollback will remove the stack
          (Choices: yes, no)

    - region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    = stack_name
          name of the cloudformation stack

    = state
          If state is "present", stack will be created.  If state is
          "present" and if stack exists and template has changed, it
          will be updated. If state is absent, stack will be removed.

    - tags
          Dictionary of tags to associate with stack and it's
          resources during stack creation. Cannot be updated later.
          Requires at least Boto version 2.6.0.

    = template
          the path of the cloudformation template

    - template_parameters
          a list of hashes of all the template variables for the stack

    Requirements:    boto

    # Basic task example
    tasks:
    - name: launch ansible cloudformation example
      action: cloudformation >
        stack_name="ansible-cloudformation" state=present
        region=us-east-1 disable_rollback=yes
        template=files/cloudformation-example.json
      args:
        template_parameters:
          KeyName: jmartin
          DiskType: ephemeral
          InstanceType: m1.small
          ClusterSize: 3
        tags:
          Stack: ansible-cloudformation

> COMMAND
=======================

::

    The [command] module takes the command name followed by a list of
    space-delimited arguments.The given command will be executed on
    all selected nodes. It will not be processed through the shell, so
    variables like `$HOME' and operations like `"<"', `">"', `"|"',
    and `"&"' will not work (use the [shell] module if you need these
    features).

    Options (= is mandatory):

    - chdir
          cd into this directory before running the command

    - creates
          a filename, when it already exists, this step will *not* be
          run.

    - executable
          change the shell used to execute the command. Should be an
          absolute path to the executable.

    = free_form
          the command module takes a free form command to run

    - removes
          a filename, when it does not exist, this step will *not* be
          run.

    Notes:    If you want to run a command through the shell (say you are using
          `<', `>', `|', etc), you actually want the [shell] module
          instead. The [command] module is much more secure as it's
          not affected by the user's environment. `creates',
          `removes', and `chdir' can be specified after the command.
          For instance, if you only want to run a command if a certain
          file does not exist, use this.

    # Example from Ansible Playbooks
    - command: /sbin/shutdown -t now
    
    # Run the command if the specified file does not exist
    - command: /usr/bin/make_database.sh arg1 arg2 creates=/path/to/database

> COPY
=======================

::

    The [copy] module copies a file on the local box to remote
    locations.

    Options (= is mandatory):

    - backup
          Create a backup file including the timestamp information so
          you can get the original file back if you somehow clobbered
          it incorrectly. (Choices: yes, no)

    - content
          When used instead of 'src', sets the contents of a file
          directly to the specified value.

    = dest
          Remote absolute path where the file should be copied to. If
          src is a directory, this must be a directory too.

    - directory_mode
          When doing a recursive copy set the mode for the
          directories. If this is not set we will default the system
          defaults.

    - force
          the default is `yes', which will replace the remote file
          when contents are different than the source.  If `no', the
          file will only be transferred if the destination does not
          exist. (Choices: yes, no)

    - others
          all arguments accepted by the [file] module also work here

    - src
          Local path to a file to copy to the remote server; can be
          absolute or relative. If path is a directory, it is copied
          recursively. In this case, if path ends with "/", only
          inside contents of that directory are copied to destination.
          Otherwise, if it does not end with "/", the directory itself
          with all contents is copied. This behavior is similar to
          Rsync.

    - validate
          The validation command to run before copying into place.
          The path to the file to validate is passed in via '%s' which
          must be present as in the visudo example below.

    Notes:    The "copy" module recursively copy facility does not scale to lots
          (>hundreds) of files. For alternative, see synchronize
          module, which is a wrapper around rsync.

    # Example from Ansible Playbooks
    - copy: src=/srv/myfiles/foo.conf dest=/etc/foo.conf owner=foo group=foo mode=0644
    
    # Copy a new "ntp.conf file into place, backing up the original if it differs from the copied version
    - copy: src=/mine/ntp.conf dest=/etc/ntp.conf owner=root group=root mode=644 backup=yes
    
    # Copy a new "sudoers" file into place, after passing validation with visudo
    - copy: src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'

> CRON
=======================

::

    Use this module to manage crontab entries. This module allows you
    to create named crontab entries, update, or delete them.The module
    includes one line with the description of the crontab entry
    `"#Ansible: <name>"' corresponding to the "name" passed to the
    module, which is used by future ansible/module calls to find/check
    the state.

    Options (= is mandatory):

    - backup
          If set, create a backup of the crontab before it is
          modified. The location of the backup is returned in the
          `backup' variable by this module.

    - cron_file
          If specified, uses this file in cron.d instead of an
          individual user's crontab.

    - day
          Day of the month the job should run ( 1-31, *, */2, etc )

    - hour
          Hour when the job should run ( 0-23, *, */2, etc )

    - job
          The command to execute. Required if state=present.

    - minute
          Minute when the job should run ( 0-59, *, */2, etc )

    - month
          Month of the year the job should run ( 1-12, *, */2, etc )

    - name
          Description of a crontab entry.

    - reboot
          If the job should be run at reboot. This option is
          deprecated. Users should use special_time. (Choices: yes,
          no)

    - special_time
          Special time specification nickname. (Choices: reboot,
          yearly, annually, monthly, weekly, daily, hourly)

    - state
          Whether to ensure the job is present or absent. (Choices:
          present, absent)

    - user
          The specific user who's crontab should be modified.

    - weekday
          Day of the week that the job should run ( 0-7 for Sunday -
          Saturday, *, etc )

    Requirements:    cron

    # Ensure a job that runs at 2 and 5 exists.
    # Creates an entry like "* 5,2 * * ls -alh > /dev/null"
    - cron: name="check dirs" hour="5,2" job="ls -alh > /dev/null"
    
    # Ensure an old job is no longer present. Removes any job that is prefixed
    # by "#Ansible: an old job" from the crontab
    - cron: name="an old job" state=absent
    
    # Creates an entry like "@reboot /some/job.sh"
    - cron: name="a job for reboot" special_time=reboot job="/some/job.sh"
    
    # Creates a cron file under /etc/cron.d
    - cron: name="yum autoupdate" weekday="2" minute=0 hour=12
            user="root" job="YUMINTERACTIVE=0 /usr/sbin/yum-autoupdate"
            cron_file=ansible_yum-autoupdate
    
    # Removes a cron file from under /etc/cron.d
    - cron: cron_file=ansible_yum-autoupdate state=absent

> DATADOG_EVENT
=======================

::

    Allows to post events to DataDog (www.datadoghq.com) service.Uses
    http://docs.datadoghq.com/api/#events API.

    Options (= is mandatory):

    - aggregation_key
          An arbitrary string to use for aggregation.

    - alert_type
          Type of alert. (Choices: error, warning, info, success)

    = api_key
          Your DataDog API key.

    - date_happened
          POSIX timestamp of the event.Default value is now.

    - priority
          The priority of the event. (Choices: normal, low)

    - tags
          Comma separated list of tags to apply to the event.

    = text
          The body of the event.

    = title
          The event title.

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Requirements:    urllib2

    # Post an event with low priority
    datadog_event: title="Testing from ansible" text="Test!" priority="low"
                   api_key="6873258723457823548234234234"
    # Post an event with several tags
    datadog_event: title="Testing from ansible" text="Test!"
                   api_key="6873258723457823548234234234"
                   tags=aa,bb,cc

> DEBUG
=======================

::

    This module prints statements during execution and can be useful
    for debugging variables or expressions without necessarily halting
    the playbook. Useful for debugging together with the 'when:'
    directive.

    Options (= is mandatory):

    - msg
          The customized message that is printed. If omitted, prints a
          generic message.

    - var
          A variable name to debug.  Mutually exclusive with the 'msg'
          option.

    # Example that prints the loopback address and gateway for each host
    - debug: msg="System {{ inventory_hostname }} has uuid {{ ansible_product_uuid }}"
    
    - debug: msg="System {{ inventory_hostname }} has gateway {{ ansible_default_ipv4.gateway }}"
      when: ansible_default_ipv4.gateway is defined
    
    - shell: /usr/bin/uptime
      register: result
    - debug: var=result
    

> DIGITAL_OCEAN
=======================

::

    Create/delete a droplet in DigitalOcean and optionally waits for
    it to be 'running', or deploy an SSH key.

    Options (= is mandatory):

    - api_key
          Digital Ocean api key.

    - client_id
          Digital Ocean manager id.

    - command
          Which target you want to operate on. (Choices: droplet, ssh)

    - id
          Numeric, the droplet id you want to operate on.

    - image_id
          Numeric, this is the id of the image you would like the
          droplet created with.

    - name
          String, this is the name of the droplet - must be formatted
          by hostname rules, or the name of a SSH key.

    - private_networking
          Bool, add an additional, private network interface to
          droplet for inter-droplet communication (Choices: yes, no)

    - region_id
          Numeric, this is the id of the region you would like your
          server

    - size_id
          Numeric, this is the id of the size you would like the
          droplet created at.

    - ssh_key_ids
          Optional, comma separated list of ssh_key_ids that you would
          like to be added to the server

    - ssh_pub_key
          The public SSH key you want to add to your account.

    - state
          Indicate desired state of the target. (Choices: present,
          active, absent, deleted)

    - unique_name
          Bool, require unique hostnames.  By default, digital ocean
          allows multiple hosts with the same name.  Setting this to
          "yes" allows only one host per name.  Useful for
          idempotence. (Choices: yes, no)

    - virtio
          Bool, turn on virtio driver in droplet for improved network
          and storage I/O (Choices: yes, no)

    - wait
          Wait for the droplet to be in state 'running' before
          returning.  If wait is "no" an ip_address may not be
          returned. (Choices: yes, no)

    - wait_timeout
          How long before wait gives up, in seconds.

    Notes:    Two environment variables can be used, DO_CLIENT_ID and
          DO_API_KEY.

    Requirements:    dopy

    # Ensure a SSH key is present
    # If a key matches this name, will return the ssh key id and changed = False
    # If no existing key matches this name, a new key is created, the ssh key id is returned and changed = False
    
    - digital_ocean: >
          state=present
          command=ssh
          name=my_ssh_key
          ssh_pub_key='ssh-rsa AAAA...'
          client_id=XXX
          api_key=XXX
    
    # Create a new Droplet
    # Will return the droplet details including the droplet id (used for idempotence)
    
    - digital_ocean: >
          state=present
          command=droplet
          name=mydroplet
          client_id=XXX
          api_key=XXX
          size_id=1
          region_id=2
          image_id=3
          wait_timeout=500
      register: my_droplet
    - debug: msg="ID is {{ my_droplet.droplet.id }}"
    - debug: msg="IP is {{ my_droplet.droplet.ip_address }}"
    
    # Ensure a droplet is present
    # If droplet id already exist, will return the droplet details and changed = False
    # If no droplet matches the id, a new droplet will be created and the droplet details (including the new id) are returned, changed = True.
    
    - digital_ocean: >
          state=present
          command=droplet
          id=123
          name=mydroplet
          client_id=XXX
          api_key=XXX
          size_id=1
          region_id=2
          image_id=3
          wait_timeout=500
    
    # Create a droplet with ssh key
    # The ssh key id can be passed as argument at the creation of a droplet (see ssh_key_ids).
    # Several keys can be added to ssh_key_ids as id1,id2,id3
    # The keys are used to connect as root to the droplet.
    
    - digital_ocean: >
          state=present
          ssh_key_ids=id1,id2
          name=mydroplet
          client_id=XXX
          api_key=XXX
          size_id=1
          region_id=2
          image_id=3

> DJANGO_MANAGE
=======================

::

    Manages a Django application using the `manage.py' application
    frontend to `django-admin'. With the `virtualenv' parameter, all
    management commands will be executed by the given `virtualenv'
    installation.

    Options (= is mandatory):

    = app_path
          The path to the root of the Django application where
          *manage.py* lives.

    - apps
          A list of space-delimited apps to target. Used by the 'test'
          command.

    - cache_table
          The name of the table used for database-backed caching. Used
          by the 'createcachetable' command.

    = command
          The name of the Django management command to run. Allowed
          commands are cleanup, createcachetable, flush, loaddata,
          syncdb, test, validate. (Choices: cleanup, flush, loaddata,
          runfcgi, syncdb, test, validate, migrate, collectstatic)

    - database
          The database to target. Used by the 'createcachetable',
          'flush', 'loaddata', and 'syncdb' commands.

    - failfast
          Fail the command immediately if a test fails. Used by the
          'test' command. (Choices: yes, no)

    - fixtures
          A space-delimited list of fixture file names to load in the
          database. *Required* by the 'loaddata' command.

    - link
          Will create links to the files instead of copying them, you
          can only use this parameter with 'collectstatic' command

    - merge
          Will run out-of-order or missing migrations as they are not
          rollback migrations, you can only use this parameter with
          'migrate' command

    - pythonpath
          A directory to add to the Python path. Typically used to
          include the settings module if it is located external to the
          application directory.

    - settings
          The Python path to the application's settings module, such
          as 'myapp.settings'.

    - skip
          Will skip over out-of-order missing migrations, you can only
          use this parameter with `migrate'

    - virtualenv
          An optional path to a `virtualenv' installation to use while
          running the manage application.

    Notes:    `virtualenv' (http://www.virtualenv.org) must be installed on the
          remote host if the virtualenv parameter is specified.This
          module will create a virtualenv if the virtualenv parameter
          is specified and a virtualenv does not already exist at the
          given location.This module assumes English error messages
          for the 'createcachetable' command to detect table
          existence, unfortunately.To be able to use the migrate
          command, you must have south installed and added as an app
          in your settingsTo be able to use the collectstatic command,
          you must have enabled staticfiles in your settings

    Requirements:    virtualenv, django

    # Run cleanup on the application installed in 'django_dir'.
    - django_manage: command=cleanup app_path={{ django_dir }}
    
    # Load the initial_data fixture into the application
    - django_manage: command=loaddata app_path={{ django_dir }} fixtures={{ initial_data }}
    
    #Run syncdb on the application
    - django_manage: >
          command=syncdb
          app_path={{ django_dir }}
          settings={{ settings_app_name }}
          pythonpath={{ settings_dir }}
          virtualenv={{ virtualenv_dir }}
    
    #Run the SmokeTest test case from the main app. Useful for testing deploys.
    - django_manage: command=test app_path=django_dir apps=main.SmokeTest

> DNSMADEEASY
=======================

::

    Manages DNS records via the v2 REST API of the DNS Made Easy
    service.  It handles records only; there is no manipulation of
    domains or monitor/account support yet. See:
    http://www.dnsmadeeasy.com/services/rest-api/

    Options (= is mandatory):

    = account_key
          Accout API Key.

    = account_secret
          Accout Secret Key.

    = domain
          Domain to work with. Can be the domain name (e.g.
          "mydomain.com") or the numeric ID of the domain in DNS Made
          Easy (e.g. "839989") for faster resolution.

    - record_name
          Record name to get/create/delete/update. If record_name is
          not specified; all records for the domain will be returned
          in "result" regardless of the state argument.

    - record_ttl
          record's "Time to live".  Number of seconds the record
          remains cached in DNS servers.

    - record_type
          Record type. (Choices: A, AAAA, CNAME, HTTPRED, MX, NS, PTR,
          SRV, TXT)

    - record_value
          Record value. HTTPRED: <redirection URL>, MX: <priority>
          <target name>, NS: <name server>, PTR: <target name>, SRV:
          <priority> <weight> <port> <target name>, TXT: <text
          value>If record_value is not specified; no changes will be
          made and the record will be returned in 'result' (in other
          words, this module can be used to fetch a record's current
          id, type, and ttl)

    = state
          whether the record should exist or not (Choices: present,
          absent)

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Notes:    The DNS Made Easy service requires that machines interacting with
          the API have the proper time and timezone set. Be sure you
          are within a few seconds of actual time by using NTP.This
          module returns record(s) in the "result" element when
          'state' is set to 'present'. This value can be be registered
          and used in your playbooks.

    Requirements:    urllib, urllib2, hashlib, hmac

    # fetch my.com domain records
    - dnsmadeeasy: account_key=key account_secret=secret domain=my.com state=present
      register: response
      
    # create / ensure the presence of a record
    - dnsmadeeasy: account_key=key account_secret=secret domain=my.com state=present record_name="test" record_type="A" record_value="127.0.0.1"
    
    # update the previously created record
    - dnsmadeeasy: account_key=key account_secret=secret domain=my.com state=present record_name="test" record_value="192.168.0.1"
    
    # fetch a specific record
    - dnsmadeeasy: account_key=key account_secret=secret domain=my.com state=present record_name="test"
      register: response
      
    # delete a record / ensure it is absent
    - dnsmadeeasy: account_key=key account_secret=secret domain=my.com state=absent record_name="test"

> DOCKER
=======================

::

    Manage the life cycle of docker containers.

    Options (= is mandatory):

    - command
          Set command to run in a container on startup

    - count
          Set number of containers to run

    - detach
          Enable detached mode on start up, leaves container running
          in background

    - dns
          Set custom DNS servers for the container

    - docker_url
          URL of docker host to issue commands to

    - env
          Set environment variables (e.g.
          env="PASSWORD=sEcRe7,WORKERS=4")

    - expose
          Set container ports to expose for port mappings or links.
          (If the port is already exposed using EXPOSE in a
          Dockerfile, you don't need to expose it again.)

    - hostname
          Set container hostname

    = image
          Set container image to use

    - links
          Link container(s) to other container(s) (e.g.
          links=redis,postgresql:db)

    - lxc_conf
          LXC config parameters,  e.g. lxc.aa_profile:unconfined

    - memory_limit
          Set RAM allocated to container

    - name
          Set the name of the container (cannot use with count)

    - password
          Set remote API password

    - ports
          Set private to public port mapping specification using
          docker CLI-style syntax [([<host_interface>:[host_port]])|(<
          host_port>):]<container_port>[/udp]

    - privileged
          Set whether the container should run in privileged mode

    - publish_all_ports
          Publish all exposed ports to the host interfaces

    - state
          Set the state of the container (Choices: present, stopped,
          absent, killed, restarted)

    - username
          Set remote API username

    - volumes
          Set volume(s) to mount on the container

    - volumes_from
          Set shared volume(s) from another container

    Requirements:    docker-py >= 0.3.0

    Start one docker container running tomcat in each host of the web group and bind tomcat's listening port to 8080
    on the host:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: run tomcat servers
        docker: image=centos command="service tomcat6 start" ports=8080
    
    The tomcat server's port is NAT'ed to a dynamic port on the host, but you can determine which port the server was
    mapped to using docker_containers:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: run tomcat servers
        docker: image=centos command="service tomcat6 start" ports=8080 count=5
      - name: Display IP address and port mappings for containers
        debug: msg={{inventory_hostname}}:{{item['HostConfig']['PortBindings']['8080/tcp'][0]['HostPort']}}
        with_items: docker_containers
    
    Just as in the previous example, but iterates over the list of docker containers with a sequence:
    
    - hosts: web
      sudo: yes
      vars:
        start_containers_count: 5
      tasks:
      - name: run tomcat servers
        docker: image=centos command="service tomcat6 start" ports=8080 count={{start_containers_count}}
      - name: Display IP address and port mappings for containers
        debug: msg="{{inventory_hostname}}:{{docker_containers[{{item}}]['HostConfig']['PortBindings']['8080/tcp'][0]['HostPort']}}"
        with_sequence: start=0 end={{start_containers_count - 1}}
    
    Stop, remove all of the running tomcat containers and list the exit code from the stopped containers:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: stop tomcat servers
        docker: image=centos command="service tomcat6 start" state=absent
      - name: Display return codes from stopped containers
        debug: msg="Returned {{inventory_hostname}}:{{item}}"
        with_items: docker_containers
    
    Create a named container:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: run tomcat server
        docker: image=centos name=tomcat command="service tomcat6 start" ports=8080
    
    Create multiple named containers:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: run tomcat servers
        docker: image=centos name={{item}} command="service tomcat6 start" ports=8080
        with_items:
          - crookshank
          - snowbell
          - heathcliff
          - felix
          - sylvester
    
    Create containers named in a sequence:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: run tomcat servers
        docker: image=centos name={{item}} command="service tomcat6 start" ports=8080
        with_sequence: start=1 end=5 format=tomcat_%d.example.com
    
    Create two linked containers:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: ensure redis container is running
        docker: image=crosbymichael/redis name=redis
    
      - name: ensure redis_ambassador container is running
        docker: image=svendowideit/ambassador ports=6379:6379 links=redis:redis name=redis_ambassador_ansible
    
    Create containers with options specified as key-value pairs and lists:
    
    - hosts: web
      sudo: yes
      tasks:
      - docker:
            image: namespace/image_name
            links:
              - postgresql:db
              - redis:redis
    
    
    Create containers with options specified as strings and lists as comma-separated strings:
    
    - hosts: web
      sudo: yes
      tasks:
      docker: image=namespace/image_name links=postgresql:db,redis:redis

> DOCKER_IMAGE
=======================

::

    Create, check and remove docker images

    Options (= is mandatory):

    - docker_url
          URL of docker host to issue commands to

    = name
          Image name to work with

    - nocache
          Do not use cache with building

    - path
          Path to directory with Dockerfile

    - state
          Set the state of the image (Choices: present, absent, build)

    - tag
          Image tag to work with

    - timeout
          Set image operation timeout

    Requirements:    docker-py

    Build docker image if required. Path should contains Dockerfile to build image:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: check or build image
        docker_image: path="/path/to/build/dir" name="my/app" state=present
    
    Build new version of image:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: check or build image
        docker_image: path="/path/to/build/dir" name="my/app" state=build
    
    Remove image from local docker storage:
    
    - hosts: web
      sudo: yes
      tasks:
      - name: run tomcat servers
        docker_image: name="my/app" state=absent
    

> EASY_INSTALL
=======================

::

    Installs Python libraries, optionally in a `virtualenv'

    Options (= is mandatory):

    - executable
          The explicit executable or a pathname to the executable to
          be used to run easy_install for a specific version of Python
          installed in the system. For example `easy_install-3.3', if
          there are both Python 2.7 and 3.3 installations in the
          system and you want to run easy_install for the Python 3.3
          installation.

    = name
          A Python library name

    - virtualenv
          an optional `virtualenv' directory path to install into. If
          the `virtualenv' does not exist, it is created automatically

    - virtualenv_command
          The command to create the virtual environment with. For
          example `pyvenv', `virtualenv', `virtualenv2'.

    - virtualenv_site_packages
          Whether the virtual environment will inherit packages from
          the global site-packages directory.  Note that if this
          setting is changed on an already existing virtual
          environment it will not have any effect, the environment
          must be deleted and newly created. (Choices: yes, no)

    Notes:    Please note that the [easy_install] module can only install Python
          libraries. Thus this module is not able to remove libraries.
          It is generally recommended to use the [pip] module which
          you can first install using [easy_install].Also note that
          `virtualenv' must be installed on the remote host if the
          `virtualenv' parameter is specified.

    Requirements:    virtualenv

    # Examples from Ansible Playbooks
    - easy_install: name=pip
    
    # Install Bottle into the specified virtualenv.
    - easy_install: name=bottle virtualenv=/webapps/myapp/venv

> EC2
=======================

::

    Creates or terminates ec2 instances. When created optionally waits
    for it to be 'running'. This module has a dependency on python-
    boto >= 2.5

    Options (= is mandatory):

    - assign_public_ip
          when provisioning within vpc, assign a public IP address.
          Boto library must be 2.13.0+

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - count
          number of instances to launch

    - count_tag
          Used with 'exact_count' to determine how many nodes based on
          a specific tag criteria should be running.  This can be
          expressed in multiple ways and is shown in the EXAMPLES
          section.  For instance, one can request 25 servers that are
          tagged with "class=webserver".

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints).  Must be
          specified if region is not used. If not set then the value
          of the EC2_URL environment variable, if any, is used

    - exact_count
          An integer value which indicates how many instances that
          match the 'count_tag' parameter should be running. Instances
          are either created or terminated based on this value.

    - group
          security group (or list of groups) to use with the instance

    - group_id
          security group id (or list of ids) to use with the instance

    - id
          identifier for this instance or set of instances, so that
          the module will be idempotent with respect to EC2 instances.
          This identifier is valid for at least 24 hours after the
          termination of the instance, and should not be reused for
          another call later on. For details, see the description of
          client token at http://docs.aws.amazon.com/AWSEC2/latest/Use
          rGuide/Run_Instance_Idempotency.html.

    = image
          `emi' (or `ami') to use for the instance

    - instance_ids
          list of instance ids, currently only used when
          state='absent'

    - instance_profile_name
          Name of the IAM instance profile to use. Boto library must
          be 2.5.0+

    - instance_tags
          a hash/dictionary of tags to add to the new instance;
          '{"key":"value"}' and '{"key":"value","key":"value"}'

    = instance_type
          instance type to use for the instance

    - kernel
          kernel `eki' to use for the instance

    - key_name
          key pair to use on the instance

    - monitoring
          enable detailed monitoring (CloudWatch) for instance

    - placement_group
          placement group for the instance when using EC2 Clustered
          Compute

    - private_ip
          the private ip address to assign the instance (from the vpc
          subnet)

    - ramdisk
          ramdisk `eri' to use for the instance

    - region
          The AWS region to use.  Must be specified if ec2_url is not
          used. If not specified then the value of the EC2_REGION
          environment variable, if any, is used.

    - state
          create or terminate instances

    - user_data
          opaque blob of data which is made available to the ec2
          instance

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    - volumes
          a list of volume dicts, each containing device name and
          optionally ephemeral id or snapshot id. Size and type (and
          number of iops for io device type) must be specified for a
          new volume or a root volume, and may be passed for a
          snapshot volume. For any volume, a volume size less than 1
          will be interpreted as a request not to create the volume.

    - vpc_subnet_id
          the subnet ID in which to launch the instance (VPC)

    - wait
          wait for the instance to be in state 'running' before
          returning (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    - zone
          AWS availability zone in which to launch the instance

    Requirements:    boto

    # Note: None of these examples set aws_access_key, aws_secret_key, or region.
    # It is assumed that their matching environment variables are set.
    
    # Basic provisioning example
    - local_action:
        module: ec2
        key_name: mykey
        instance_type: c1.medium
        image: emi-40603AD1
        wait: yes
        group: webserver
        count: 3
    
    # Advanced example with tagging and CloudWatch
    - local_action:
        module: ec2
        key_name: mykey
        group: databases
        instance_type: m1.large
        image: ami-6e649707
        wait: yes
        wait_timeout: 500
        count: 5
        instance_tags: 
           db: postgres
        monitoring: yes
    
    # Single instance with additional IOPS volume from snapshot
    local_action:
        module: ec2
        key_name: mykey
        group: webserver
        instance_type: m1.large
        image: ami-6e649707
        wait: yes
        wait_timeout: 500
        volumes:
        - device_name: /dev/sdb
          snapshot: snap-abcdef12
          device_type: io1
          iops: 1000
          volume_size: 100
        monitoring: yes
    
    # Multiple groups example
    local_action:
        module: ec2
        key_name: mykey
        group: ['databases', 'internal-services', 'sshable', 'and-so-forth']
        instance_type: m1.large
        image: ami-6e649707
        wait: yes
        wait_timeout: 500
        count: 5
        instance_tags: 
            db: postgres
        monitoring: yes
    
    # Multiple instances with additional volume from snapshot
    local_action:
        module: ec2
        key_name: mykey
        group: webserver
        instance_type: m1.large
        image: ami-6e649707
        wait: yes
        wait_timeout: 500
        count: 5
        volumes:
        - device_name: /dev/sdb
          snapshot: snap-abcdef12
          volume_size: 10
        monitoring: yes
    
    # VPC example
    - local_action:
        module: ec2
        key_name: mykey
        group_id: sg-1dc53f72
        instance_type: m1.small
        image: ami-6e649707
        wait: yes
        vpc_subnet_id: subnet-29e63245
        assign_public_ip: yes
    
    # Launch instances, runs some tasks
    # and then terminate them
    
    
    - name: Create a sandbox instance
      hosts: localhost
      gather_facts: False
      vars:
        key_name: my_keypair
        instance_type: m1.small
        security_group: my_securitygroup
        image: my_ami_id
        region: us-east-1
      tasks:
        - name: Launch instance
          local_action: ec2 key_name={{ keypair }} group={{ security_group }} instance_type={{ instance_type }} image={{ image }} wait=true region={{ region }}
          register: ec2
        - name: Add new instance to host group
          local_action: add_host hostname={{ item.public_ip }} groupname=launched
          with_items: ec2.instances
        - name: Wait for SSH to come up
          local_action: wait_for host={{ item.public_dns_name }} port=22 delay=60 timeout=320 state=started
          with_items: ec2.instances
    
    - name: Configure instance(s)
      hosts: launched
      sudo: True
      gather_facts: True
      roles:
        - my_awesome_role
        - my_awesome_test
    
    - name: Terminate instances
      hosts: localhost
      connection: local
      tasks:
        - name: Terminate instances that were previously launched
          local_action:
            module: ec2
            state: 'absent'
            instance_ids: '{{ ec2.instance_ids }}'
    
    # Start a few existing instances, run some tasks
    # and stop the instances
    
    - name: Start sandbox instances
      hosts: localhost
      gather_facts: false
      connection: local
      vars:
        instance_ids:
          - 'i-xxxxxx'
          - 'i-xxxxxx'
          - 'i-xxxxxx'
        region: us-east-1
      tasks:
        - name: Start the sandbox instances
          local_action:
            module: ec2
            instance_ids: '{{ instance_ids }}'
            region: '{{ region }}'
            state: running
            wait: True
      role:
        - do_neat_stuff
        - do_more_neat_stuff
    
    - name: Stop sandbox instances
      hosts: localhost
      gather_facts: false
      connection: local
      vars:
        instance_ids:
          - 'i-xxxxxx'
          - 'i-xxxxxx'
          - 'i-xxxxxx'
        region: us-east-1
      tasks:
        - name: Stop the sanbox instances
          local_action:
          module: ec2
          instance_ids: '{{ instance_ids }}'
          region: '{{ region }}'
          state: stopped
          wait: True
    
    #
    # Enforce that 5 instances with a tag "foo" are running
    #
    
    - local_action:
        module: ec2
        key_name: mykey
        instance_type: c1.medium
        image: emi-40603AD1
        wait: yes
        group: webserver
        instance_tags:
            foo: bar
        exact_count: 5
        count_tag: foo
    
    #
    # Enforce that 5 running instances named "database" with a "dbtype" of "postgres"
    #
    
    - local_action:
        module: ec2
        key_name: mykey
        instance_type: c1.medium
        image: emi-40603AD1
        wait: yes
        group: webserver
        instance_tags: 
            Name: database
            dbtype: postgres
        exact_count: 5
        count_tag: 
            Name: database
            dbtype: postgres
    
    #
    # count_tag complex argument examples
    #
    
        # instances with tag foo
        count_tag:
            foo:
    
        # instances with tag foo=bar
        count_tag:
            foo: bar
    
        # instances with tags foo=bar & baz
        count_tag:
            foo: bar
            baz:
    
        # instances with tags foo & bar & baz=bang
        count_tag:
            - foo
            - bar
            - baz: bang
    

> EC2_AMI
=======================

::

    Creates or deletes ec2 images. This module has a dependency on
    python-boto >= 2.5

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - delete_snapshot
          Whether or not to deleted an AMI while deregistering it.

    - description
          An optional human-readable string describing the contents
          and purpose of the AMI.

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints).  Must be
          specified if region is not used. If not set then the value
          of the EC2_URL environment variable, if any, is used

    - image_id
          Image ID to be deregistered.

    - instance_id
          instance id of the image to create

    - name
          The name of the new image to create

    - no_reboot
          An optional flag indicating that the bundling process should
          not attempt to shutdown the instance before bundling. If
          this flag is True, the responsibility of maintaining file
          system integrity is left to the owner of the instance. The
          default choice is "no". (Choices: yes, no)

    - region
          The AWS region to use.  Must be specified if ec2_url is not
          used. If not specified then the value of the EC2_REGION
          environment variable, if any, is used.

    - state
          create or deregister/delete image

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    - wait
          wait for the AMI to be in state 'available' before
          returning. (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    Requirements:    boto

    # Basic AMI Creation
    - local_action:
        module: ec2_ami
        aws_access_key: xxxxxxxxxxxxxxxxxxxxxxx
        aws_secret_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        instance_id: i-xxxxxx
        wait: yes
        name: newtest
      register: instance
    
    # Basic AMI Creation, without waiting
    - local_action:
        module: ec2_ami
        aws_access_key: xxxxxxxxxxxxxxxxxxxxxxx
        aws_secret_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        region: xxxxxx
        instance_id: i-xxxxxx
        wait: no
        name: newtest
      register: instance
    
    # Deregister/Delete AMI
    - local_action:
        module: ec2_ami
        aws_access_key: xxxxxxxxxxxxxxxxxxxxxxx
        aws_secret_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        region: xxxxxx
        image_id: ${instance.image_id}
        delete_snapshot: True
        state: absent
    
    # Deregister AMI
    - local_action:
        module: ec2_ami
        aws_access_key: xxxxxxxxxxxxxxxxxxxxxxx
        aws_secret_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        region: xxxxxx
        image_id: ${instance.image_id}
        delete_snapshot: False
        state: absent
    

> EC2_EIP
=======================

::

    This module associates AWS EC2 elastic IP addresses with instances

    Options (= is mandatory):

    - ec2_access_key
          EC2 access key. If not specified then the EC2_ACCESS_KEY
          environment variable is used.

    - ec2_secret_key
          EC2 secret key. If not specified then the EC2_SECRET_KEY
          environment variable is used.

    - ec2_url
          URL to use to connect to EC2-compatible cloud (by default
          the module will use EC2 endpoints)

    - in_vpc
          allocate an EIP inside a VPC or not

    - instance_id
          The EC2 instance id

    - public_ip
          The elastic IP address to associate with the instance.If
          absent, allocate a new address

    - region
          the EC2 region to use

    - state
          If present, associate the IP with the instance.If absent,
          disassociate the IP with the instance. (Choices: present,
          absent)

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    Notes:    This module will return `public_ip' on success, which will contain
          the public IP address associated with the instance.There may
          be a delay between the time the Elastic IP is assigned and
          when the cloud instance is reachable via the new address.
          Use wait_for and pause to delay further playbook execution
          until the instance is reachable, if necessary.

    Requirements:    boto

    - name: associate an elastic IP with an instance
      ec2_eip: instance_id=i-1212f003 ip=93.184.216.119
    
    - name: disassociate an elastic IP from an instance
      ec2_eip: instance_id=i-1212f003 ip=93.184.216.119 state=absent
    
    - name: allocate a new elastic IP and associate it with an instance
      ec2_eip: instance_id=i-1212f003
    
    - name: allocate a new elastic IP without associating it to anything
      ec2_eip:
      register: eip
    - name: output the IP
      debug: msg="Allocated IP is {{ eip.public_ip }}"
    
    - name: provision new instances with ec2
      ec2: keypair=mykey instance_type=c1.medium image=emi-40603AD1 wait=yes group=webserver count=3
      register: ec2
    - name: associate new elastic IPs with each of the instances
      ec2_eip: "instance_id={{ item }}"
      with_items: ec2.instance_ids
    
    - name: allocate a new elastic IP inside a VPC in us-west-2
      ec2_eip: region=us-west-2 in_vpc=yes
      register: eip
    - name: output the IP
      debug: msg="Allocated IP inside a VPC is {{ eip.public_ip }}"

> EC2_ELB
=======================

::

    This module de-registers or registers an AWS EC2 instance from the
    ELBs that it belongs to.Returns fact "ec2_elbs" which is a list of
    elbs attached to the instance if state=absent is passed as an
    argument.Will be marked changed when called only if there are ELBs
    found to operate on.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - ec2_elbs
          List of ELB names, required for registration. The ec2_elbs
          fact should be used if there was a previous de-register.

    - enable_availability_zone
          Whether to enable the availability zone of the instance on
          the target ELB if the availability zone has not already been
          enabled. If set to no, the task will fail if the
          availability zone is not enabled on the ELB. (Choices: yes,
          no)

    = instance_id
          EC2 Instance ID

    - region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    = state
          register or deregister the instance (Choices: present,
          absent)

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    - wait
          Wait for instance registration or deregistration to complete
          successfully before returning. (Choices: yes, no)

    Requirements:    boto

    # basic pre_task and post_task example
    pre_tasks:
      - name: Gathering ec2 facts
        ec2_facts:
      - name: Instance De-register
        local_action: ec2_elb
        args:
          instance_id: "{{ ansible_ec2_instance_id }}"
          state: 'absent'
    roles:
      - myrole
    post_tasks:
      - name: Instance Register
        local_action: ec2_elb
        args:
          instance_id: "{{ ansible_ec2_instance_id }}"
          ec2_elbs: "{{ item }}"
          state: 'present'
        with_items: ec2_elbs

> EC2_ELB_LB
=======================

::

    Creates or destroys Amazon ELB.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - health_check
          An associative array of health check configuration settigs
          (see example)

    - listeners
          List of ports/protocols for this ELB to listen on (see
          example)

    = name
          The name of the ELB

    - purge_listeners
          Purge existing listeners on ELB that are not found in
          listeners

    - purge_zones
          Purge existing availability zones on ELB that are not found
          in zones

    - region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    = state
          Create or destroy the ELB

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    - zones
          List of availability zones to enable on this ELB

    Requirements:    boto

    # Note: None of these examples set aws_access_key, aws_secret_key, or region.
    # It is assumed that their matching environment variables are set.
    
    # Basic provisioning example
    - local_action:
        module: ec2_elb_lb
        name: "test-please-delete"
        state: present
        zones:
          - us-east-1a
          - us-east-1d
        listeners:
          - protocol: http # options are http, https, ssl, tcp
            load_balancer_port: 80
            instance_port: 80
          - protocol: https
            load_balancer_port: 443
            instance_protocol: http # optional, defaults to value of protocol setting
            instance_port: 80
            # ssl certificate required for https or ssl
            ssl_certificate_id: "arn:aws:iam::123456789012:server-certificate/company/servercerts/ProdServerCert"
    
    # Configure a health check
    - local_action:
        module: ec2_elb_lb
        name: "test-please-delete"
        state: present
        zones:
          - us-east-1d
        listeners:
          - protocol: http
            load_balancer_port: 80
            instance_port: 80
        health_check:
            ping_protocol: http # options are http, https, ssl, tcp
            ping_port: 80
            ping_path: "/index.html" # not required for tcp or ssl
            response_timeout: 5 # seconds
            interval: 30 # seconds
            unhealthy_threshold: 2
            healthy_threshold: 10
    
    # Ensure ELB is gone
    - local_action:
        module: ec2_elb_lb
        name: "test-please-delete"
        state: absent
    
    # Normally, this module will purge any listeners that exist on the ELB
    # but aren't specified in the listeners parameter. If purge_listeners is
    # false it leaves them alone
    - local_action:
        module: ec2_elb_lb
        name: "test-please-delete"
        state: present
        zones:
          - us-east-1a
          - us-east-1d
        listeners:
          - protocol: http
            load_balancer_port: 80
            instance_port: 80
        purge_listeners: no
    
    # Normally, this module will leave availability zones that are enabled
    # on the ELB alone. If purge_zones is true, then any extreneous zones
    # will be removed
    - local_action:
        module: ec2_elb_lb
        name: "test-please-delete"
        state: present
        zones:
          - us-east-1a
          - us-east-1d
        listeners:
          - protocol: http
            load_balancer_port: 80
            instance_port: 80
        purge_zones: yes

> EC2_FACTS
=======================

::

    This module fetches data from the metadata servers in ec2 (aws).
    Eucalyptus cloud provides a similar service and this module should
    work this cloud provider as well.

    Options (= is mandatory):

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Notes:    Parameters to filter on ec2_facts may be added later.

    # Conditional example
    - name: Gather facts
      action: ec2_facts
    
    - name: Conditional
      action: debug msg="This instance is a t1.micro"
      when: ansible_ec2_instance_type == "t1.micro"

> EC2_GROUP
=======================

::

    maintains ec2 security groups. This module has a dependency on
    python-boto >= 2.5

    Options (= is mandatory):

    = description
          Description of the security group.

    - ec2_access_key
          EC2 access key

    - ec2_secret_key
          EC2 secret key

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints)

    = name
          Name of the security group.

    - region
          the EC2 region to use

    = rules
          List of firewall rules to enforce in this group (see
          example).

    - state
          create or delete security group

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    - vpc_id
          ID of the VPC to create the group in.

    Requirements:    boto

    - name: example ec2 group
      local_action:
        module: ec2_group
        name: example
        description: an example EC2 group
        vpc_id: 12345
        region: eu-west-1a
        ec2_secret_key: SECRET
        ec2_access_key: ACCESS
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 10.0.0.0/8
          - proto: udp
            from_port: 10050
            to_port: 10050
            cidr_ip: 10.0.0.0/8
          - proto: udp
            from_port: 10051
            to_port: 10051
            group_id: sg-12345678
          - proto: all
            # the containing group name may be specified here
            group_name: example

> EC2_KEY
=======================

::

    maintains ec2 key pairs. This module has a dependency on python-
    boto >= 2.5

    Options (= is mandatory):

    - ec2_access_key
          EC2 access key

    - ec2_secret_key
          EC2 secret key

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints)

    - key_material
          Public key material.

    = name
          Name of the key pair.

    - region
          the EC2 region to use

    - state
          create or delete keypair

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    Requirements:    boto

    # Note: None of these examples set aws_access_key, aws_secret_key, or region.
    # It is assumed that their matching environment variables are set.
    
    # Creates a new ec2 key pair named `example` if not present, returns generated
    # private key
    - name: example ec2 key
      local_action:
        module: ec2_key
        name: example
    
    # Creates a new ec2 key pair named `example` if not present using provided key
    # material
    - name: example2 ec2 key
      local_action:
        module: ec2_key
        name: example2
        key_material: 'ssh-rsa AAAAxyz...== me@example.com'
        state: present
    
    # Creates a new ec2 key pair named `example` if not present using provided key
    # material
    - name: example3 ec2 key
      local_action:
        module: ec2_key
        name: example3
        key_material: "{{ item }}"
      with_file: /path/to/public_key.id_rsa.pub
    
    # Removes ec2 key pair by name
    - name: remove example key
      local_action:
        module: ec2_key
        name: example
        state: absent

> EC2_SNAPSHOT
=======================

::

    creates an EC2 snapshot from an existing EBS volume

    Options (= is mandatory):

    - description
          description to be applied to the snapshot

    - device_name
          device name of a mounted volume to be snapshotted

    - ec2_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - ec2_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints).  Must be
          specified if region is not used. If not set then the value
          of the EC2_URL environment variable, if any, is used

    - instance_id
          instance that has a the required volume to snapshot mounted

    - region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    - volume_id
          volume from which to take the snapshot

    Requirements:    boto

    # Simple snapshot of volume using volume_id
    - local_action: 
        module: ec2_snapshot 
        volume_id: vol-abcdef12   
        description: snapshot of /data from DB123 taken 2013/11/28 12:18:32
    
    # Snapshot of volume mounted on device_name attached to instance_id
    - local_action: 
        module: ec2_snapshot 
        instance_id: i-12345678
        device_name: /dev/sdb1
        description: snapshot of /data from DB123 taken 2013/11/28 12:18:32

> EC2_TAG
=======================

::

    Creates and removes tags from any EC2 resource.  The resource is
    referenced by its resource id (e.g. an instance being i-XXXXXXX).
    It is designed to be used with complex args (tags), see the
    examples.  This module has a dependency on python-boto.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints).  Must be
          specified if region is not used. If not set then the value
          of the EC2_URL environment variable, if any, is used.

    - region
          region in which the resource exists.

    = resource
          The EC2 resource id.

    - state
          Whether the tags should be present or absent on the
          resource. (Choices: present, absent)

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    Requirements:    boto

    # Basic example of adding tag(s)
    tasks:
    - name: tag a resource
      local_action: ec2_tag resource=vol-XXXXXX region=eu-west-1 state=present
      args:
        tags:
          Name: ubervol
          env: prod
    
    # Playbook example of adding tag(s) to spawned instances
    tasks:
    - name: launch some instances
      local_action: ec2 keypair={{ keypair }} group={{ security_group }} instance_type={{ instance_type }} image={{ image_id }} wait=true region=eu-west-1
      register: ec2
    
    - name: tag my launched instances
      local_action: ec2_tag resource={{ item.id }} region=eu-west-1 state=present
      with_items: ec2.instances
      args:
        tags:
          Name: webserver
          env: prod

> EC2_VOL
=======================

::

    creates an EBS volume and optionally attaches it to an instance.
    If both an instance ID and a device name is given and the instance
    has a device at the device name, then no volume is created and no
    attachment is made.  This module has a dependency on python-boto.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - device_name
          device id to override device mapping. Assumes /dev/sdf for
          Linux/UNIX and /dev/xvdf for Windows.

    - ec2_url
          Url to use to connect to EC2 or your Eucalyptus cloud (by
          default the module will use EC2 endpoints).  Must be
          specified if region is not used. If not set then the value
          of the EC2_URL environment variable, if any, is used

    - instance
          instance ID if you wish to attach the volume.

    - iops
          the provisioned IOPs you want to associate with this volume
          (integer).

    - region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    - snapshot
          snapshot ID on which to base the volume

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    = volume_size
          size of volume (in GB) to create.

    - zone
          zone in which to create the volume, if unset uses the zone
          the instance is in (if set)

    Requirements:    boto

    # Simple attachment action
    - local_action: 
        module: ec2_vol 
        instance: XXXXXX 
        volume_size: 5 
        device_name: sdd
    
    # Example using custom iops params   
    - local_action: 
        module: ec2_vol 
        instance: XXXXXX 
        volume_size: 5 
        iops: 200
        device_name: sdd
    
    # Example using snapshot id
    - local_action:
        module: ec2_vol
        instance: XXXXXX
        snapshot: "{{ snapshot }}"
    
    # Playbook example combined with instance launch 
    - local_action: 
        module: ec2 
        keypair: "{{ keypair }}"
        image: "{{ image }}"
        wait: yes 
        count: 3
        register: ec2
    - local_action: 
        module: ec2_vol 
        instance: "{{ item.id }} " 
        volume_size: 5
        with_items: ec2.instances
        register: ec2_vol

> EC2_VPC
=======================

::

    Create or terminates AWS virtual private clouds.  This module has
    a dependency on python-boto.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    = cidr_block
          The cidr block representing the VPC, e.g. 10.0.0.0/16

    - dns_hostnames
          toggles the "Enable DNS hostname support for instances" flag
          (Choices: yes, no)

    - dns_support
          toggles the "Enable DNS resolution" flag (Choices: yes, no)

    - instance_tenancy
          The supported tenancy options for instances launched into
          the VPC. (Choices: default, dedicated)

    - internet_gateway
          Toggle whether there should be an Internet gateway attached
          to the VPC (Choices: yes, no)

    - region
          region in which the resource exists.

    - route_tables
          A dictionary array of route tables to add of the form: {
          subnets: [172.22.2.0/24, 172.22.3.0/24,], routes: [{ dest:
          0.0.0.0/0, gw: igw},] }. Where the subnets list is those
          subnets the route table should be associated with, and the
          routes list is a list of routes to be in the table.  The
          special keyword for the gw of igw specifies that you should
          the route should go through the internet gateway attached to
          the VPC. gw also accepts instance-ids in addition igw. This
          module is currently unable to affect the 'main' route table
          due to some limitations in boto, so you must explicitly
          define the associated subnets or they will be attached to
          the main table implicitly.

    = state
          Create or terminate the VPC

    - subnets
          A dictionary array of subnets to add of the form: { cidr:
          ..., az: ... }. Where az is the desired availability zone of
          the subnet, but it is not required. All VPC subnets not in
          this list will be removed.

    - validate_certs
          When set to "no", SSL certificates will not be validated for
          boto versions >= 2.6.0. (Choices: yes, no)

    - vpc_id
          A VPC id to terminate when state=absent

    - wait
          wait for the VPC to be in state 'available' before returning
          (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    Requirements:    boto

    # Note: None of these examples set aws_access_key, aws_secret_key, or region.
    # It is assumed that their matching environment variables are set.
    
    # Basic creation example:
          local_action:
            module: ec2_vpc
            state: present
            cidr_block: 172.23.0.0/16
            region: us-west-2
    # Full creation example with subnets and optional availability zones.
    # The absence or presense of subnets deletes or creates them respectively.
          local_action:
            module: ec2_vpc
            state: present
            cidr_block: 172.22.0.0/16
            subnets: 
              - cidr: 172.22.1.0/24
                az: us-west-2c
              - cidr: 172.22.2.0/24
                az: us-west-2b
              - cidr: 172.22.3.0/24
                az: us-west-2a
            internet_gateway: True
            route_tables:
              - subnets: 
                  - 172.22.2.0/24
                  - 172.22.3.0/24
                routes: 
                  - dest: 0.0.0.0/0
                    gw: igw
              - subnets:
                  - 172.22.1.0/24
                routes:
                  - dest: 0.0.0.0/0
                    gw: igw
            region: us-west-2
          register: vpc
    
    # Removal of a VPC by id
          local_action:
            module: ec2_vpc
            state: absent
            vpc_id: vpc-aaaaaaa 
            region: us-west-2  
    If you have added elements not managed by this module, e.g. instances, NATs, etc then
    the delete will fail until those dependencies are removed.

> EJABBERD_USER
=======================

::

    This module provides user management for ejabberd servers

    Options (= is mandatory):

    = host
          the ejabberd host associated with this username

    - logging
          enables or disables the local syslog facility for this
          module (Choices: true, false, yes, no)

    - password
          the password to assign to the username

    - state
          describe the desired state of the user to be managed
          (Choices: present, absent)

    = username
          the name of the user to manage

    Notes:    Password parameter is required for state == present onlyPasswords
          must be stored in clear text for this release

    Requirements:    ejabberd

    Example playbook entries using the ejabberd_user module to manage users state.
    
        tasks:
    
        - name: create a user if it does not exists
          action: ejabberd_user username=test host=server password=password
    
        - name: delete a user if it exists
          action: ejabberd_user username=test host=server state=absent

> ELASTICACHE
=======================

::

    Manage cache clusters in Amazon Elasticache.Returns information
    about the specified cache cluster.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - cache_engine_version
          The version number of the cache engine

    - cache_port
          The port number on which each of the cache nodes will accept
          connections

    - cache_security_groups
          A list of cache security group names to associate with this
          cache cluster

    - engine
          Name of the cache engine to be used (memcached or redis)

    - hard_modify
          Whether to destroy and recreate an existing cache cluster if
          necessary in order to modify its state (Choices: yes, no)

    = name
          The cache cluster identifier

    - node_type
          The compute and memory capacity of the nodes in the cache
          cluster

    - num_nodes
          The initial number of cache nodes that the cache cluster
          will have

    - region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    = state
          `absent' or `present' are idempotent actions that will
          create or destroy a cache cluster as needed. `rebooted' will
          reboot the cluster, resulting in a momentary outage.
          (Choices: present, absent, rebooted)

    - wait
          Wait for cache cluster result before returning (Choices:
          yes, no)

    - zone
          The EC2 Availability Zone in which the cache cluster will be
          created

    Requirements:    boto

    # Note: None of these examples set aws_access_key, aws_secret_key, or region.
    # It is assumed that their matching environment variables are set.
    
    # Basic example
    - local_action:
        module: elasticache
        name: "test-please-delete"
        state: present
        engine: memcached
        cache_engine_version: 1.4.14
        node_type: cache.m1.small
        num_nodes: 1
        cache_port: 11211
        cache_security_groups:
          - default
        zone: us-east-1d
    
    
    # Ensure cache cluster is gone
    - local_action:
        module: elasticache
        name: "test-please-delete"
        state: absent
    
    # Reboot cache cluster
    - local_action:
        module: elasticache
        name: "test-please-delete"
        state: rebooted
    

> FACTER
=======================

::

    Runs the `facter' discovery program
    (https://github.com/puppetlabs/facter) on the remote system,
    returning JSON data that can be useful for inventory purposes.

    Requirements:    facter, ruby-json

    # Example command-line invocation
    ansible www.example.net -m facter

> FAIL
=======================

::

    This module fails the progress with a custom message. It can be
    useful for bailing out when a certain condition is met using
    `when'.

    Options (= is mandatory):

    - msg
          The customized message used for failing execution. If
          omitted, fail will simple bail out with a generic message.

    # Example playbook using fail and when together
    - fail: msg="The system may not be provisioned according to the CMDB status."
      when: cmdb_status != "to-be-staged"

> FETCH
=======================

::

    This module works like [copy], but in reverse. It is used for
    fetching files from remote machines and storing them locally in a
    file tree, organized by hostname. Note that this module is written
    to transfer log files that might not be present, so a missing
    remote file won't be an error unless fail_on_missing is set to
    'yes'.

    Options (= is mandatory):

    = dest
          A directory to save the file into. For example, if the
          `dest' directory is `/backup' a `src' file named
          `/etc/profile' on host `host.example.com', would be saved
          into `/backup/host.example.com/etc/profile'

    - fail_on_missing
          Makes it fails when the source file is missing. (Choices:
          yes, no)

    - flat
          Allows you to override the default behavior of prepending
          hostname/path/to/file to the destination.  If dest ends with
          '/', it will use the basename of the source file, similar to
          the copy module.  Obviously this is only handy if the
          filenames are unique.

    = src
          The file on the remote system to fetch. This `must' be a
          file, not a directory. Recursive fetching may be supported
          in a later release.

    - validate_md5
          Verify that the source and destination md5sums match after
          the files are fetched. (Choices: yes, no)

    # Store file into /tmp/fetched/host.example.com/tmp/somefile
    - fetch: src=/tmp/somefile dest=/tmp/fetched
    
    # Specifying a path directly
    - fetch: src=/tmp/somefile dest=/tmp/prefix-{{ ansible_hostname }} flat=yes
    
    # Specifying a destination path
    - fetch: src=/tmp/uniquefile dest=/tmp/special/ flat=yes
    
    # Storing in a path relative to the playbook
    - fetch: src=/tmp/uniquefile dest=special/prefix-{{ ansible_hostname }} flat=yes

> FILE
=======================

::

    Sets attributes of files, symlinks, and directories, or removes
    files/symlinks/directories. Many other modules support the same
    options as the [file] module - including [copy], [template], and
    [assemble].

    Options (= is mandatory):

    - force
          force the creation of the symlinks in two cases: the source
          file does not exist (but will appear later); the destination
          exists and is a file (so, we need to unlink the "path" file
          and create symlink to the "src" file in place of it).
          (Choices: yes, no)

    - group
          name of the group that should own the file/directory, as
          would be fed to `chown' (Choices: )

    - mode
          mode the file or directory should be, such as 0644 as would
          be fed to `chmod' (Choices: )

    - owner
          name of the user that should own the file/directory, as
          would be fed to `chown' (Choices: )

    = path
          path to the file being managed.  Aliases: `dest', `name'

    - recurse
          recursively set the specified file attributes (applies only
          to state=directory) (Choices: yes, no)

    - selevel
          level part of the SELinux file context. This is the MLS/MCS
          attribute, sometimes known as the `range'. `_default'
          feature works as for `seuser'. (Choices: )

    - serole
          role part of SELinux file context, `_default' feature works
          as for `seuser'. (Choices: )

    - setype
          type part of SELinux file context, `_default' feature works
          as for `seuser'. (Choices: )

    - seuser
          user part of SELinux file context. Will default to system
          policy, if applicable. If set to `_default', it will use the
          `user' portion of the policy if available (Choices: )

    - src
          path of the file to link to (applies only to `state=link').
          Will accept absolute, relative and nonexisting paths.
          Relative paths are not expanded. (Choices: )

    - state
          If `directory', all immediate subdirectories will be created
          if they do not exist. If `file', the file will NOT be
          created if it does not exist, see the [copy] or [template]
          module if you want that behavior. If `link', the symbolic
          link will be created or changed. Use `hard' for hardlinks.
          If `absent', directories will be recursively deleted, and
          files or symlinks will be unlinked. If `touch' (new in 1.4),
          an empty file will be created if the c(dest) does not exist,
          while an existing file or directory will receive updated
          file access and modification times (similar to the way
          `touch` works from the command line). (Choices: file, link,
          directory, hard, touch, absent)

    Notes:    See also [copy], [template], [assemble]

    - file: path=/etc/foo.conf owner=foo group=foo mode=0644
    - file: src=/file/to/link/to dest=/path/to/symlink owner=foo group=foo state=link

> FILESYSTEM
=======================

::

    This module creates file system.

    Options (= is mandatory):

    = dev
          Target block device.

    - force
          If yes, allows to create new filesystem on devices that
          already has filesystem. (Choices: yes, no)

    = fstype
          File System type to be created.

    - opts
          List of options to be passed to mkfs command.

    Notes:    uses mkfs command

    # Create a ext2 filesystem on /dev/sdb1.
    - filesystem: fstype=ext2 dev=/dev/sdb1
    
    # Create a ext4 filesystem on /dev/sdb1 and check disk blocks.
    - filesystem: fstype=ext4 dev=/dev/sdb1 opts="-cc"

> FIREBALL
=======================

::

    This modules launches an ephemeral `fireball' ZeroMQ message bus
    daemon on the remote node which Ansible can use to communicate
    with nodes at high speed.The daemon listens on a configurable port
    for a configurable amount of time.Starting a new fireball as a
    given user terminates any existing user fireballs.Fireball mode is
    AES encrypted

    Options (= is mandatory):

    - minutes
          The `fireball' listener daemon is started on nodes and will
          stay around for this number of minutes before turning itself
          off.

    - port
          TCP port for ZeroMQ

    Notes:    See the advanced playbooks chapter for more about using fireball
          mode.

    Requirements:    zmq, keyczar

    # This example playbook has two plays: the first launches 'fireball' mode on all hosts via SSH, and 
    # the second actually starts using it for subsequent management over the fireball connection
    
    - hosts: devservers
      gather_facts: false
      connection: ssh
      sudo: yes
      tasks:
          - action: fireball
    
    - hosts: devservers
      connection: fireball
      tasks:
          - command: /usr/bin/anything

> FIREWALLD
=======================

::

    This module allows for addition or deletion of services and ports
    either tcp or udp in either running or permanent firewalld rules

    Options (= is mandatory):

    = permanent
          Should this configuration be in the running firewalld
          configuration or persist across reboots

    - port
          Name of a port to add/remove to/from firewalld must be in
          the form PORT/PROTOCOL

    - rich_rule
          Rich rule to add/remove to/from firewalld

    - service
          Name of a service to add/remove to/from firewalld - service
          must be listed in /etc/services

    = state
          Should this port accept(enabled) or reject(disabled)
          connections

    - timeout
          The amount of time the rule should be in effect for when
          non-permanent

    - zone
          The firewalld zone to add/remove to/from (NOTE: default zone
          can be configured per system but "public" is default from
          upstream. Available choices can be extended based on per-
          system configs, listed here are "out of the box" defaults).
          (Choices: work, drop, internal, external, trusted, home,
          dmz, public, block)

    Notes:    Not tested on any debian based system

    Requirements:    firewalld >= 0.2.11

    - firewalld: service=https permanent=true state=enabled
    - firewalld: port=8081/tcp permanent=true state=disabled
    - firewalld: zone=dmz service=http permanent=true state=enabled
    - firewalld: rich_rule='rule service name="ftp" audit limit value="1/m" accept' permanent=true state=enabled

> FLOWDOCK
=======================

::

    Send a message to a flowdock team inbox or chat using the push API
    (see https://www.flowdock.com/api/team-inbox and
    https://www.flowdock.com/api/chat)

    Options (= is mandatory):

    - external_user_name
          (chat only - required) Name of the "user" sending the
          message

    - from_address
          (inbox only - required) Email address of the message sender

    - from_name
          (inbox only) Name of the message sender

    - link
          (inbox only) Link associated with the message. This will be
          used to link the message subject in Team Inbox.

    = msg
          Content of the message

    - project
          (inbox only) Human readable identifier for more detailed
          message categorization

    - reply_to
          (inbox only) Email address for replies

    - source
          (inbox only - required) Human readable identifier of the
          application that uses the Flowdock API

    - subject
          (inbox only - required) Subject line of the message

    - tags
          tags of the message, separated by commas

    = token
          API token.

    = type
          Whether to post to 'inbox' or 'chat' (Choices: inbox, chat)

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Requirements:    urllib, urllib2

    - flowdock: type=inbox
                token=AAAAAA
                from_address=user@example.com
                source='my cool app'
                msg='test from ansible'
                subject='test subject'
    
    - flowdock: type=chat
                token=AAAAAA
                external_user_name=testuser
                msg='test from ansible'
                tags=tag1,tag2,tag3

> GC_STORAGE
=======================

::

    This module allows users to manage their objects/buckets in Google
    Cloud Storage.  It allows upload and download operations and can
    set some canned permissions. It also allows retrieval of URLs for
    objects for use in playbooks, and retrieval of string contents of
    objects.  This module requires setting the default project in GCS
    prior to playbook usage.  See https://developers.google.com/storag
    e/docs/reference/v1/apiversion1 for information about setting the
    default project.

    Options (= is mandatory):

    = bucket
          Bucket name.

    - dest
          The destination file path when downloading an object/key
          with a GET operation.

    - expiration
          Time limit (in seconds) for the URL generated and returned
          by GCA when performing a mode=put or mode=get_url operation.
          This url is only avaialbe when public-read is the acl for
          the object.

    - force
          Forces an overwrite either locally on the filesystem or
          remotely with the object/key. Used with PUT and GET
          operations.

    = gcs_access_key
          GCS access key. If not set then the value of the
          GCS_ACCESS_KEY environment variable is used.

    = gcs_secret_key
          GCS secret key. If not set then the value of the
          GCS_SECRET_KEY environment variable is used.

    = mode
          Switches the module behaviour between upload, download,
          get_url (return download url) , get_str (download object as
          string), create (bucket) and delete (bucket). (Choices: get,
          put, get_url, get_str, delete, create)

    - object
          Keyname of the object inside the bucket. Can be also be used
          to create "virtual directories" (see examples).

    - permission
          This option let's the user set the canned permissions on the
          object/bucket that are created. The permissions that can be
          set are 'private', 'public-read', 'authenticated-read'.

    - src
          The source file path when performing a PUT operation.

    Requirements:    boto 2.9+

    # upload some content
    - gc_storage: bucket=mybucket object=key.txt src=/usr/local/myfile.txt mode=put permission=public-read
    
    # download some content
    - gc_storage: bucket=mybucket object=key.txt dest=/usr/local/myfile.txt mode=get
    
    # Download an object as a string to use else where in your playbook
    - gc_storage: bucket=mybucket object=key.txt mode=get_str
    
    # Create an empty bucket
    - gc_storage: bucket=mybucket mode=create
    
    # Create a bucket with key as directory
    - gc_storage: bucket=mybucket object=/my/directory/path mode=create
    
    # Delete a bucket and all contents
    - gc_storage: bucket=mybucket mode=delete

> GCE
=======================

::

    Creates or terminates Google Compute Engine (GCE) instances.  See
    https://cloud.google.com/products/compute-engine for an overview.
    Full install/configuration instructions for the gce* modules can
    be found in the comments of ansible/test/gce_tests.py.

    Options (= is mandatory):

    - image
          image string to use for the instance

    - instance_names
          a comma-separated list of instance names to create or
          destroy

    - machine_type
          machine type to use for the instance, use 'n1-standard-1' by
          default

    - metadata
          a hash/dictionary of custom data for the instance;
          '{"key":"value",...}'

    - name
          identifier when working with a single instance

    - network
          name of the network, 'default' will be used if not specified

    - persistent_boot_disk
          if set, create the instance with a persistent boot disk

    - state
          desired state of the resource (Choices: active, present,
          absent, deleted)

    - tags
          a comma-separated list of tags to associate with the
          instance

    = zone
          the GCE zone to use (Choices: us-central1-a, us-central1-b,
          us-central2-a, europe-west1-a, europe-west1-b)

    Requirements:    libcloud

    # Basic provisioning example.  Create a single Debian 7 instance in the
    # us-central1-a Zone of n1-standard-1 machine type.
    - local_action:
        module: gce
        name: test-instance
        zone: us-central1-a
        machine_type: n1-standard-1
        image: debian-7
    
    # Example using defaults and with metadata to create a single 'foo' instance
    - local_action:
        module: gce
        name: foo
        metadata: '{"db":"postgres", "group":"qa", "id":500}'
    
    
    # Launch instances from a control node, runs some tasks on the new instances,
    # and then terminate them
    - name: Create a sandbox instance
      hosts: localhost
      vars:
        names: foo,bar
        machine_type: n1-standard-1
        image: debian-6
        zone: us-central1-a
      tasks:
        - name: Launch instances
          local_action: gce instance_names={{names}} machine_type={{machine_type}}
                        image={{image}} zone={{zone}}
          register: gce
        - name: Wait for SSH to come up
          local_action: wait_for host={{item.public_ip}} port=22 delay=10
                        timeout=60 state=started
          with_items: {{gce.instance_data}}
    
    - name: Configure instance(s)
      hosts: launched
      sudo: True
      roles:
        - my_awesome_role
        - my_awesome_tasks
    
    - name: Terminate instances
      hosts: localhost
      connection: local
      tasks:
        - name: Terminate instances that were previously launched
          local_action:
            module: gce
            state: 'absent'
            instance_names: {{gce.instance_names}}
    

> GCE_LB
=======================

::

    This module can create and destroy Google Compute Engine
    `loadbalancer' and `httphealthcheck' resources.  The primary LB
    resource is the `load_balancer' resource and the health check
    parameters are all prefixed with `httphealthcheck'. The full
    documentation for Google Compute Engine load balancing is at
    https://developers.google.com/compute/docs/load-balancing/.
    However, the ansible module simplifies the configuration by
    following the libcloud model. Full install/configuration
    instructions for the gce* modules can be found in the comments of
    ansible/test/gce_tests.py.

    Options (= is mandatory):

    - external_ip
          the external static IPv4 (or auto-assigned) address for the
          LB

    - httphealthcheck_healthy_count
          number of consecutive successful checks before marking a
          node healthy

    - httphealthcheck_host
          host header to pass through on HTTP check requests

    - httphealthcheck_interval
          the duration in seconds between each health check request

    - httphealthcheck_name
          the name identifier for the HTTP health check

    - httphealthcheck_path
          the url path to use for HTTP health checking

    - httphealthcheck_port
          the TCP port to use for HTTP health checking

    - httphealthcheck_timeout
          the timeout in seconds before a request is considered a
          failed check

    - httphealthcheck_unhealthy_count
          number of consecutive failed checks before marking a node
          unhealthy

    - members
          a list of zone/nodename pairs, e.g ['us-central1-a/www-a',
          ...]

    - name
          name of the load-balancer resource

    - port_range
          the port (range) to forward, e.g. 80 or 8000-8888 defaults
          to all ports

    - protocol
          the protocol used for the load-balancer packet forwarding,
          tcp or udp (Choices: tcp, udp)

    - region
          the GCE region where the load-balancer is defined (Choices:
          us-central1, us-central2, europe-west1)

    - state
          desired state of the LB (Choices: active, present, absent,
          deleted)

    Requirements:    libcloud

    # Simple example of creating a new LB, adding members, and a health check
    - local_action: 
        module: gce_lb
        name: testlb
        region: us-central1
        members: ["us-central1-a/www-a", "us-central1-b/www-b"]
        httphealthcheck_name: hc
        httphealthcheck_port: 80
        httphealthcheck_path: "/up"

> GCE_NET
=======================

::

    This module can create and destroy Google Compue Engine networks
    and firewall rules
    https://developers.google.com/compute/docs/networking. The `name'
    parameter is reserved for referencing a network while the `fwname'
    parameter is used to reference firewall rules. IPv4 Address ranges
    must be specified using the CIDR http://en.wikipedia.org/wiki
    /Classless_Inter-Domain_Routing format. Full install/configuration
    instructions for the gce* modules can be found in the comments of
    ansible/test/gce_tests.py.

    Options (= is mandatory):

    - allowed
          the protocol:ports to allow ('tcp:80' or 'tcp:80,443' or
          'tcp:80-800')

    - fwname
          name of the firewall rule

    - ipv4_range
          the IPv4 address range in CIDR notation for the network

    - name
          name of the network

    - src_range
          the source IPv4 address range in CIDR notation

    - src_tags
          the source instance tags for creating a firewall rule

    - state
          desired state of the persistent disk (Choices: active,
          present, absent, deleted)

    Requirements:    libcloud

    # Simple example of creating a new network
    - local_action: 
        module: gce_net
        name: privatenet
        ipv4_range: '10.240.16.0/24'
       
    # Simple example of creating a new firewall rule
    - local_action: 
        module: gce_net
        name: privatenet
        allowed: tcp:80,8080
        src_tags: ["web", "proxy"]
    

> GCE_PD
=======================

::

    This module can create and destroy unformatted GCE persistent
    disks
    https://developers.google.com/compute/docs/disks#persistentdisks.
    It also supports attaching and detaching disks from running
    instances but does not support creating boot disks from images or
    snapshots.  The 'gce' module supports creating instances with boot
    disks. Full install/configuration instructions for the gce*
    modules can be found in the comments of ansible/test/gce_tests.py.

    Options (= is mandatory):

    - detach_only
          do not destroy the disk, merely detach it from an instance
          (Choices: yes, no)

    - instance_name
          instance name if you wish to attach or detach the disk

    - mode
          GCE mount mode of disk, READ_ONLY (default) or READ_WRITE
          (Choices: READ_WRITE, READ_ONLY)

    = name
          name of the disk

    - size_gb
          whole integer size of disk (in GB) to create, default is 10
          GB

    - state
          desired state of the persistent disk (Choices: active,
          present, absent, deleted)

    - zone
          zone in which to create the disk

    Requirements:    libcloud

    # Simple attachment action to an existing instance
    - local_action: 
        module: gce_pd 
        instance_name: notlocalhost
        size_gb: 5 
        name: pd

> GEM
=======================

::

    Manage installation and uninstallation of Ruby gems.

    Options (= is mandatory):

    - executable
          Override the path to the gem executable

    - gem_source
          The path to a local gem used as installation source.

    - include_dependencies
          Wheter to include dependencies or not. (Choices: yes, no)

    = name
          The name of the gem to be managed.

    - repository
          The repository from which the gem will be installed

    = state
          The desired state of the gem. `latest' ensures that the
          latest version is installed. (Choices: present, absent,
          latest)

    - user_install
          Install gem in user's local gems cache or for all users

    - version
          Version of the gem to be installed/removed.

    # Installs version 1.0 of vagrant.
    - gem: name=vagrant version=1.0 state=present
    
    # Installs latest available version of rake.
    - gem: name=rake state=latest
    
    # Installs rake version 1.0 from a local gem on disk.
    - gem: name=rake gem_source=/path/to/gems/rake-1.0.gem state=present

> GET_URL
=======================

::

    Downloads files from HTTP, HTTPS, or FTP to the remote server. The
    remote server `must' have direct access to the remote resource.By
    default, if an environment variable `<protocol>_proxy' is set on
    the target host, requests will be sent through that proxy. This
    behaviour can be overridden by setting a variable for this task
    (see `setting the environment
    <http://docs.ansible.com/playbooks_environment.html>`_), or by
    using the use_proxy option.

    Options (= is mandatory):

    = dest
          absolute path of where to download the file to.If `dest' is
          a directory, either the server provided filename or, if none
          provided, the base name of the URL on the remote server will
          be used. If a directory, `force' has no effect. If `dest' is
          a directory, the file will always be downloaded (regardless
          of the force option), but replaced only if the contents
          changed.

    - force
          If `yes' and `dest' is not a directory, will download the
          file every time and replace the file if the contents change.
          If `no', the file will only be downloaded if the destination
          does not exist. Generally should be `yes' only for small
          local files. Prior to 0.6, this module behaved as if `yes'
          was the default. (Choices: yes, no)

    - others
          all arguments accepted by the [file] module also work here

    - sha256sum
          If a SHA-256 checksum is passed to this parameter, the
          digest of the destination file will be calculated after it
          is downloaded to ensure its integrity and verify that the
          transfer completed successfully.

    = url
          HTTP, HTTPS, or FTP URL in the form
          (http|https|ftp)://[user[:pass]]@host.domain[:port]/path

    - use_proxy
          if `no', it will not use a proxy, even if one is defined in
          an environment variable on the target hosts. (Choices: yes,
          no)

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Notes:    This module doesn't yet support configuration for proxies.

    Requirements:    urllib2, urlparse

    - name: download foo.conf
      get_url: url=http://example.com/path/file.conf dest=/etc/foo.conf mode=0440
    
    - name: download file with sha256 check
      get_url: url=http://example.com/path/file.conf dest=/etc/foo.conf sha256sum=b5bb9d8014a0f9b1d61e21e796d78dccdf1352f23cd32812f4850b878ae4944c

> GIT
=======================

::

    Manage `git' checkouts of repositories to deploy files or
    software.

    Options (= is mandatory):

    - accept_hostkey
          Add the hostkey for the repo url if not already added. If
          ssh_args contains "-o StrictHostKeyChecking=no", this
          parameter is ignored.

    - bare
          if `yes', repository will be created as a bare repo,
          otherwise it will be a standard repo with a workspace.
          (Choices: yes, no)

    - depth
          Create a shallow clone with a history truncated to the
          specified number or revisions. The minimum possible value is
          `1', otherwise ignored.

    = dest
          Absolute path of where the repository should be checked out
          to.

    - executable
          Path to git executable to use. If not supplied, the normal
          mechanism for resolving binary paths will be used.

    - force
          If `yes', any modified files in the working repository will
          be discarded.  Prior to 0.7, this was always 'yes' and could
          not be disabled. (Choices: yes, no)

    - key_file
          Uses the same wrapper method as ssh_opts to pass "-i
          <key_file>" to the ssh arguments used by git

    - reference
          Reference repository (see "git clone --reference ...")

    - remote
          Name of the remote.

    = repo
          git, SSH, or HTTP protocol address of the git repository.

    - ssh_opts
          Creates a wrapper script and exports the path as GIT_SSH
          which git then automatically uses to override ssh arguments.
          An example value could be "-o StrictHostKeyChecking=no"

    - update
          If `yes', repository will be updated using the supplied
          remote.  Otherwise the repo will be left untouched. Prior to
          1.2, this was always 'yes' and could not be disabled.
          (Choices: yes, no)

    - version
          What version of the repository to check out.  This can be
          the full 40-character `SHA-1' hash, the literal string
          `HEAD', a branch name, or a tag name.

    Notes:    If the task seems to be hanging, first verify remote host is in
          `known_hosts'. SSH will prompt user to authorize the first
          contact with a remote host.  To avoid this prompt, one
          solution is to add the remote host public key in
          `/etc/ssh/ssh_known_hosts' before calling the git module,
          with the following command: ssh-keyscan remote_host.com >>
          /etc/ssh/ssh_known_hosts.

    # Example git checkout from Ansible Playbooks
    - git: repo=git://foosball.example.org/path/to/repo.git
           dest=/srv/checkout
           version=release-0.22
    
    # Example read-write git checkout from github
    - git: repo=ssh://git@github.com/mylogin/hello.git dest=/home/mylogin/hello
    
    # Example just ensuring the repo checkout exists
    - git: repo=git://foosball.example.org/path/to/repo.git dest=/srv/checkout update=no

> GITHUB_HOOKS
=======================

::

    Adds service hooks and removes service hooks that have an error
    status.

    Options (= is mandatory):

    = action
          This tells the githooks module what you want it to do.
          (Choices: create, cleanall)

    - hookurl
          When creating a new hook, this is the url that you want
          github to post to. It is only required when creating a new
          hook.

    = oauthkey
          The oauth key provided by github. It can be found/generated
          on github under "Edit Your Profile" >> "Applications" >>
          "Personal Access Tokens"

    = repo
          This is the API url for the repository you want to manage
          hooks for. It should be in the form of:
          https://api.github.com/repos/user:/repo:. Note this is
          different than the normal repo url.

    = user
          Github username.

    - validate_certs
          If `no', SSL certificates for the target repo will not be
          validated. This should only be used on personally controlled
          sites using self-signed certificates. (Choices: yes, no)

    # Example creating a new service hook. It ignores duplicates.
    - github_hooks: action=create hookurl=http://11.111.111.111:2222 user={{ gituser }} oauthkey={{ oauthkey }} repo=https://api.github.com/repos/pcgentry/Github-Auto-Deploy
    
    # Cleaning all hooks for this repo that had an error on the last update. Since this works for all hooks in a repo it is probably best that this would be called from a handler.
    - local_action: github_hooks action=cleanall user={{ gituser }} oauthkey={{ oauthkey }} repo={{ repo }}

> GLANCE_IMAGE
=======================

::

    Add or Remove images from the glance repository.

    Options (= is mandatory):

    - auth_url
          The keystone url for authentication

    - container_format
          The format of the container

    - copy_from
          A url from where the image can be downloaded, mutually
          exclusive with file parameter

    - disk_format
          The format of the disk that is getting uploaded

    - file
          The path to the file which has to be uploaded, mutually
          exclusive with copy_from

    - is_public
          Whether the image can be accessed publicly

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    - min_disk
          The minimum disk space required to deploy this image

    - min_ram
          The minimum ram required to deploy this image

    = name
          Name that has to be given to the image

    - owner
          The owner of the image

    - region_name
          Name of the region

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - timeout
          The time to wait for the image process to complete in
          seconds

    Requirements:    glanceclient, keystoneclient

    # Upload an image from an HTTP URL
    - glance_image: login_username=admin
                    login_password=passme
                    login_tenant_name=admin
                    name=cirros
                    container_format=bare 
                    disk_format=qcow2
                    state=present
                    copy_from=http:launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img

> GROUP
=======================

::

    Manage presence of groups on a host.

    Options (= is mandatory):

    - gid
          Optional `GID' to set for the group.

    = name
          Name of the group to manage.

    - state
          Whether the group should be present or not on the remote
          host. (Choices: present, absent)

    - system
          If `yes', indicates that the group created is a system
          group. (Choices: yes, no)

    Requirements:    groupadd, groupdel, groupmod

    # Example group command from Ansible Playbooks
    - group: name=somegroup state=present

> GROUP_BY
=======================

::

    Use facts to create ad-hoc groups that can be used later in a
    playbook.

    Options (= is mandatory):

    = key
          The variables whose values will be used as groups

    Notes:    Spaces in group names are converted to dashes '-'.

    # Create groups based on the machine architecture
    -  group_by: key=machine_{{ ansible_machine }}
    # Create groups like 'kvm-host'
    -  group_by: key=virt_{{ ansible_virtualization_type }}_{{ ansible_virtualization_role }}

> GROVE
=======================

::

    The [grove] module sends a message for a service to a Grove.io
    channel.

    Options (= is mandatory):

    = channel_token
          Token of the channel to post to.

    - icon_url
          Icon for the service

    = message
          Message content

    - service
          Name of the service (displayed as the "user" in the message)

    - url
          Service URL for the web client

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    - grove: >
        channel_token=6Ph62VBBJOccmtTPZbubiPzdrhipZXtg
        service=my-app
        message=deployed {{ target }}

> HG
=======================

::

    Manages Mercurial (hg) repositories. Supports SSH, HTTP/S and
    local address.

    Options (= is mandatory):

    = dest
          Absolute path of where the repository should be cloned to.

    - executable
          Path to hg executable to use. If not supplied, the normal
          mechanism for resolving binary paths will be used.

    - force
          Discards uncommitted changes. Runs `hg update -C'. (Choices:
          yes, no)

    - purge
          Deletes untracked files. Runs `hg purge'. (Choices: yes, no)

    = repo
          The repository address.

    - revision
          Equivalent `-r' option in hg command which could be the
          changeset, revision number, branch name or even tag.

    Notes:    If the task seems to be hanging, first verify remote host is in
          `known_hosts'. SSH will prompt user to authorize the first
          contact with a remote host.  To avoid this prompt, one
          solution is to add the remote host public key in
          `/etc/ssh/ssh_known_hosts' before calling the hg module,
          with the following command: ssh-keyscan remote_host.com >>
          /etc/ssh/ssh_known_hosts.

    # Ensure the current working copy is inside the stable branch and deletes untracked files if any.
    - hg: repo=https://bitbucket.org/user/repo1 dest=/home/user/repo1 revision=stable purge=yes

> HIPCHAT
=======================

::

    Send a message to hipchat

    Options (= is mandatory):

    - color
          Background color for the message. Default is yellow.
          (Choices: yellow, red, green, purple, gray, random)

    - from
          Name the message will appear be sent from. max 15
          characters. Over 15, will be shorten.

    = msg
          The message body.

    - msg_format
          message format. html or text. Default is text. (Choices:
          text, html)

    - notify
          notify or not (change the tab color, play a sound, etc)
          (Choices: yes, no)

    = room
          ID or name of the room.

    = token
          API token.

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Requirements:    urllib, urllib2

    - hipchat: token=AAAAAA room=notify msg="Ansible task finished"

> HOMEBREW
=======================

::

    Manages Homebrew packages

    Options (= is mandatory):

    - install_options
          options flags to install a package

    = name
          name of package to install/remove

    - state
          state of the package (Choices: present, absent)

    - update_homebrew
          update homebrew itself first (Choices: yes, no)

    - homebrew: name=foo state=present
    - homebrew: name=foo state=present update_homebrew=yes
    - homebrew: name=foo state=absent
    - homebrew: name=foo,bar state=absent
    - homebrew: name=foo state=present install_options=with-baz,enable-debug

> HOSTNAME
=======================

::

    Set system's hostnameCurrently implemented on only Debian, Ubuntu,
    RedHat and CentOS.

    Options (= is mandatory):

    = name
          Name of the host

    Requirements:    hostname

    - hostname: name=web01

> HTPASSWD
=======================

::

    Add and remove username/password entries in a password file using
    htpasswd.This is used by web servers such as Apache and Nginx for
    basic authentication.

    Options (= is mandatory):

    - create
          Used with `state=present'. If specified, the file will be
          created if it does not already exist. If set to "no", will
          fail if the file does not exist (Choices: yes, no)

    - crypt_scheme
          Encryption scheme to be used. (Choices: apr_md5_crypt,
          des_crypt, ldap_sha1, plaintext)

    = name
          User name to add or remove

    - password
          Password associated with user.Must be specified if user does
          not exist yet.

    = path
          Path to the file that contains the usernames and passwords

    - state
          Whether the user entry should be present or not (Choices:
          present, absent)

    Notes:    This module depends on the `passlib' Python library, which needs
          to be installed on all target systems.On Debian, Ubuntu, or
          Fedora: install `python-passlib'.On RHEL or CentOS: Enable
          EPEL, then install `python-passlib'.

    # Add a user to a password file and ensure permissions are set
    - htpasswd: path=/etc/nginx/passwdfile name=janedoe password=9s36?;fyNp owner=root group=www-data mode=0640
    # Remove a user from a password file
    - htpasswd: path=/etc/apache2/passwdfile name=foobar state=absent

> INCLUDE_VARS
=======================

::

    Loads variables from a YAML file dynamically during task runtime.
    It can work with conditionals, or use host specific variables to
    determine the path name to load from.

    Options (= is mandatory):

    = free-form
          The file name from which variables should be loaded, if
          called from a role it will look for the file in vars/
          subdirectory of the role, otherwise the path would be
          relative to playbook. An absolute path can also be provided.

    # Conditionally decide to load in variables when x is 0, otherwise do not.
    - include_vars: contingency_plan.yml
      when: x == 0
    
    # Load a variable file based on the OS type, or a default if not found.
    - include_vars: "{{ item }}"
      with_first_found:
       - "{{ ansible_os_distribution }}.yml"
       - "default.yml"
     

> INI_FILE
=======================

::

    Manage (add, remove, change) individual settings in an INI-style
    file without having to manage the file as a whole with, say,
    [template] or [assemble]. Adds missing sections if they don't
    exist.Comments are discarded when the source file is read, and
    therefore will not show up in the destination file.

    Options (= is mandatory):

    - backup
          Create a backup file including the timestamp information so
          you can get the original file back if you somehow clobbered
          it incorrectly. (Choices: yes, no)

    = dest
          Path to the INI-style file; this file is created if required

    - option
          if set (required for changing a `value'), this is the name
          of the option.May be omitted if adding/removing a whole
          `section'.

    - others
          all arguments accepted by the [file] module also work here

    = section
          Section name in INI file. This is added if `state=present'
          automatically when a single value is being set.

    - value
          the string value to be associated with an `option'. May be
          omitted when removing an `option'.

    Notes:    While it is possible to add an `option' without specifying a
          `value', this makes no sense.A section named `default'
          cannot be added by the module, but if it exists, individual
          options within the section can be updated. (This is a
          limitation of Python's `ConfigParser'.) Either use
          [template] to create a base INI file with a `[default]'
          section, or use [lineinfile] to add the missing line.

    Requirements:    ConfigParser

    # Ensure "fav=lemonade is in section "[drinks]" in specified file
    - ini_file: dest=/etc/conf section=drinks option=fav value=lemonade mode=0600 backup=yes
    
    - ini_file: dest=/etc/anotherconf
                section=drinks
                option=temperature
                value=cold
                backup=yes

> IRC
=======================

::

    Send a message to an IRC channel. This is a very simplistic
    implementation.

    Options (= is mandatory):

    = channel
          Channel name

    - color
          Text color for the message. Default is black. (Choices:
          yellow, red, green, blue, black)

    = msg
          The message body.

    - nick
          Nickname

    - passwd
          Server password

    - port
          IRC server port number

    - server
          IRC server name/address

    - timeout
          Timeout to use while waiting for successful registration and
          join messages, this is to prevent an endless loop

    Requirements:    socket

    - irc: server=irc.example.net channel="#t1" msg="Hello world"
    
    - local_action: irc port=6669
                    channel="#t1"
                    msg="All finished at {{ ansible_date_time.iso8601 }}"
                    color=red
                    nick=ansibleIRC

> JABBER
=======================

::

    Send a message to jabber

    Options (= is mandatory):

    - encoding
          message encoding

    - host
          host to connect, overrides user info

    = msg
          The message body.

    = password
          password for user to connect

    - port
          port to connect to, overrides default

    = to
          user ID or name of the room, when using room use a slash to
          indicate your nick.

    = user
          User as which to connect

    Requirements:    xmpp

    # send a message to a user
    - jabber: user=mybot@example.net
              password=secret
              to=friend@example.net
              msg="Ansible task finished"
    
    # send a message to a room
    - jabber: user=mybot@example.net
              password=secret
              to=mychaps@conference.example.net/ansiblebot
              msg="Ansible task finished"
    
    # send a message, specifying the host and port
    - jabber user=mybot@example.net
             host=talk.example.net
             port=5223
             password=secret
             to=mychaps@example.net
             msg="Ansible task finished"

> JBOSS
=======================

::

    Deploy applications to JBoss standalone using the filesystem

    Options (= is mandatory):

    - deploy_path
          The location in the filesystem where the deployment scanner
          listens

    = deployment
          The name of the deployment

    - src
          The remote path of the application ear or war to deploy

    - state
          Whether the application should be deployed or undeployed
          (Choices: present, absent)

    Notes:    The JBoss standalone deployment-scanner has to be enabled in
          standalone.xmlEnsure no identically named application is
          deployed through the JBoss CLI

    # Deploy a hello world application
    - jboss: src=/tmp/hello-1.0-SNAPSHOT.war deployment=hello.war state=present
    # Update the hello world application
    - jboss: src=/tmp/hello-1.1-SNAPSHOT.war deployment=hello.war state=present
    # Undeploy the hello world application
    - jboss: deployment=hello.war state=absent

> KERNEL_BLACKLIST
=======================

::

    Add or remove kernel modules from blacklist.

    Options (= is mandatory):

    - blacklist_file
          If specified, use this blacklist file instead of
          `/etc/modprobe.d/blacklist-ansible.conf'.

    = name
          Name of kernel module to black- or whitelist.

    - state
          Whether the module should be present in the blacklist or
          absent. (Choices: present, absent)

    # Blacklist the nouveau driver module
    - kernel_blacklist: name=nouveau state=present

> KEYSTONE_USER
=======================

::

    Manage users,tenants, roles from OpenStack.

    Options (= is mandatory):

    - description
          A description for the tenant

    - email
          An email address for the user

    - endpoint
          The keystone url for authentication

    - login_password
          Password of login user

    - login_tenant_name
          The tenant login_user belongs to

    - login_user
          login username to authenticate to keystone

    - password
          The password to be assigned to the user

    - role
          The name of the role to be assigned or created

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - tenant
          The tenant name that has be added/removed

    - token
          The token to be uses in case the password is not specified

    - user
          The name of the user that has to added/removed from
          OpenStack

    Requirements:    python-keystoneclient

    # Create a tenant
    - keystone_user: tenant=demo tenant_description="Default Tenant"
    
    # Create a user
    - keystone_user: user=john tenant=demo password=secrete
    
    # Apply the admin role to the john user in the demo tenant
    - keystone_user: role=admin user=john tenant=demo

> LINEINFILE
=======================

::

    This module will search a file for a line, and ensure that it is
    present or absent.This is primarily useful when you want to change
    a single line in a file only. For other cases, see the [copy] or
    [template] modules.

    Options (= is mandatory):

    - backrefs
          Used with `state=present'. If set, line can contain
          backreferences (both positional and named) that will get
          populated if the `regexp' matches. This flag changes the
          operation of the module slightly; `insertbefore' and
          `insertafter' will be ignored, and if the `regexp' doesn't
          match anywhere in the file, the file will be left unchanged.
          If the `regexp' does match, the last matching line will be
          replaced by the expanded line parameter. (Choices: yes, no)

    - backup
          Create a backup file including the timestamp information so
          you can get the original file back if you somehow clobbered
          it incorrectly. (Choices: yes, no)

    - create
          Used with `state=present'. If specified, the file will be
          created if it does not already exist. By default it will
          fail if the file is missing. (Choices: yes, no)

    = dest
          The file to modify.

    - insertafter
          Used with `state=present'. If specified, the line will be
          inserted after the specified regular expression. A special
          value is available; `EOF' for inserting the line at the end
          of the file. May not be used with `backrefs'. (Choices: EOF,
          *regex*)

    - insertbefore
          Used with `state=present'. If specified, the line will be
          inserted before the specified regular expression. A value is
          available; `BOF' for inserting the line at the beginning of
          the file. May not be used with `backrefs'. (Choices: BOF,
          *regex*)

    - line
          Required for `state=present'. The line to insert/replace
          into the file. If `backrefs' is set, may contain
          backreferences that will get expanded with the `regexp'
          capture groups if the regexp matches. The backreferences
          should be double escaped (see examples).

    - others
          All arguments accepted by the [file] module also work here.

    - regexp
          The regular expression to look for in every line of the
          file. For `state=present', the pattern to replace if found;
          only the last line found will be replaced. For
          `state=absent', the pattern of the line to remove.  Uses
          Python regular expressions; see
          http://docs.python.org/2/library/re.html.

    - state
          Whether the line should be there or not. (Choices: present,
          absent)

    - validate
          validation to run before copying into place

    - lineinfile: dest=/etc/selinux/config regexp=^SELINUX= line=SELINUX=disabled
    
    - lineinfile: dest=/etc/sudoers state=absent regexp="^%wheel"
    
    - lineinfile: dest=/etc/hosts regexp='^127\.0\.0\.1' line='127.0.0.1 localhost' owner=root group=root mode=0644
    
    - lineinfile: dest=/etc/httpd/conf/httpd.conf regexp="^Listen " insertafter="^#Listen " line="Listen 8080"
    
    - lineinfile: dest=/etc/services regexp="^# port for http" insertbefore="^www.*80/tcp" line="# port for http by default"
    
    # Add a line to a file if it does not exist, without passing regexp
    - lineinfile: dest=/tmp/testfile line="192.168.1.99 foo.lab.net foo"
    
    # Fully quoted because of the ': ' on the line. See the Gotchas in the YAML docs.
    - lineinfile: "dest=/etc/sudoers state=present regexp='^%wheel' line='%wheel ALL=(ALL) NOPASSWD: ALL'"
    
    - lineinfile: dest=/opt/jboss-as/bin/standalone.conf regexp='^(.*)Xms(\d+)m(.*)$' line='\1Xms${xms}m\3' backrefs=yes
    
    # Validate a the sudoers file before saving
    - lineinfile: dest=/etc/sudoers state=present regexp='^%ADMIN ALL\=' line='%ADMIN ALL=(ALL) NOPASSWD:ALL' validate='visudo -cf %s'

> LINODE
=======================

::

    creates / deletes a Linode Public Cloud instance and optionally
    waits for it to be 'running'.

    Options (= is mandatory):

    - api_key
          Linode API key

    - datacenter
          datacenter to create an instance in (Linode Datacenter)

    - distribution
          distribution to use for the instance (Linode Distribution)

    - linode_id
          Unique ID of a linode server

    - name
          Name to give the instance (alphanumeric, dashes,
          underscore)To keep sanity on the Linode Web Console, name is
          prepended with LinodeID_

    - password
          root password to apply to a new server (auto generated if
          missing)

    - payment_term
          payment term to use for the instance (payment term in
          months) (Choices: 1, 12, 24)

    - plan
          plan to use for the instance (Linode plan)

    - ssh_pub_key
          SSH public key applied to root user

    - state
          Indicate desired state of the resource (Choices: present,
          active, started, absent, deleted, stopped, restarted)

    - swap
          swap size in MB

    - wait
          wait for the instance to be in state 'running' before
          returning (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    Notes:    LINODE_API_KEY env variable can be used instead

    Requirements:    linode-python

    # Create a server
    - local_action:
         module: linode
         api_key: 'longStringFromLinodeApi'
         name: linode-test1
         plan: 1
         datacenter: 2
         distribution: 99
         password: 'superSecureRootPassword'
         ssh_pub_key: 'ssh-rsa qwerty'
         swap: 768
         wait: yes
         wait_timeout: 600
         state: present
    
    # Ensure a running server (create if missing)
    - local_action:
         module: linode
         api_key: 'longStringFromLinodeApi'
         name: linode-test1
         linode_id: 12345678
         plan: 1
         datacenter: 2
         distribution: 99
         password: 'superSecureRootPassword'
         ssh_pub_key: 'ssh-rsa qwerty'
         swap: 768
         wait: yes
         wait_timeout: 600
         state: present
    
    # Delete a server
    - local_action:
         module: linode
         api_key: 'longStringFromLinodeApi'
         name: linode-test1
         linode_id: 12345678
         state: absent
    
    # Stop a server
    - local_action:
         module: linode
         api_key: 'longStringFromLinodeApi'
         name: linode-test1
         linode_id: 12345678
         state: stopped
    
    # Reboot a server
    - local_action:
         module: linode
         api_key: 'longStringFromLinodeApi'
         name: linode-test1
         linode_id: 12345678
         state: restarted

> LVG
=======================

::

    This module creates, removes or resizes volume groups.

    Options (= is mandatory):

    - force
          If yes, allows to remove volume group with logical volumes.
          (Choices: yes, no)

    - pesize
          The size of the physical extent in megabytes. Must be a
          power of 2.

    - pvs
          List of comma-separated devices to use as physical devices
          in this volume group. Required when creating or resizing
          volume group.

    - state
          Control if the volume group exists. (Choices: present,
          absent)

    = vg
          The name of the volume group.

    Notes:    module does not modify PE size for already present volume group

    # Create a volume group on top of /dev/sda1 with physical extent size = 32MB.
    - lvg:  vg=vg.services pvs=/dev/sda1 pesize=32
    
    # Create or resize a volume group on top of /dev/sdb1 and /dev/sdc5. 
    # If, for example, we already have VG vg.services on top of /dev/sdb1,
    # this VG will be extended by /dev/sdc5.  Or if vg.services was created on
    # top of /dev/sda5, we first extend it with /dev/sdb1 and /dev/sdc5,
    # and then reduce by /dev/sda5.
    - lvg: vg=vg.services pvs=/dev/sdb1,/dev/sdc5
    
    # Remove a volume group with name vg.services.
    - lvg: vg=vg.services state=absent

> LVOL
=======================

::

    This module creates, removes or resizes logical volumes.

    Options (= is mandatory):

    - force
          Shrink or remove operations of volumes requires this switch.
          Ensures that that filesystems get never corrupted/destroyed
          by mistake. (Choices: yes, no)

    = lv
          The name of the logical volume.

    - size
          The size of the logical volume, according to lvcreate(8)
          --size, by default in megabytes or optionally with one of
          [bBsSkKmMgGtTpPeE] units; or according to lvcreate(8)
          --extents as a percentage of [VG|PVS|FREE]; resizing is not
          supported with percentages.

    - state
          Control if the logical volume exists. (Choices: present,
          absent)

    = vg
          The volume group this logical volume is part of.

    Notes:    Filesystems on top of the volume are not resized.

    # Create a logical volume of 512m.
    - lvol: vg=firefly lv=test size=512
    
    # Create a logical volume of 512g.
    - lvol: vg=firefly lv=test size=512g
    
    # Create a logical volume the size of all remaining space in the volume group
    - lvol: vg=firefly lv=test size=100%FREE
    
    # Extend the logical volume to 1024m.
    - lvol: vg=firefly lv=test size=1024
    
    # Reduce the logical volume to 512m
    - lvol: vg=firefly lv=test size=512 force=yes
    
    # Remove the logical volume.
    - lvol: vg=firefly lv=test state=absent force=yes

> MACPORTS
=======================

::

    Manages MacPorts packages

    Options (= is mandatory):

    = name
          name of package to install/remove

    - state
          state of the package (Choices: present, absent, active,
          inactive)

    - update_cache
          update the package db first (Choices: yes, no)

    - macports: name=foo state=present
    - macports: name=foo state=present update_cache=yes
    - macports: name=foo state=absent
    - macports: name=foo state=active
    - macports: name=foo state=inactive

> MAIL
=======================

::

    This module is useful for sending emails from playbooks.One may
    wonder why automate sending emails?  In complex environments there
    are from time to time processes that cannot be automated, either
    because you lack the authority to make it so, or because not
    everyone agrees to a common approach.If you cannot automate a
    specific step, but the step is non-blocking, sending out an email
    to the responsible party to make him perform his part of the
    bargain is an elegant way to put the responsibility in someone
    else's lap.Of course sending out a mail can be equally useful as a
    way to notify one or more people in a team that a specific action
    has been (successfully) taken.

    Options (= is mandatory):

    - attach
          A space-separated list of pathnames of files to attach to
          the message. Attached files will have their content-type set
          to `application/octet-stream'.

    - bcc
          The email-address(es) the mail is being 'blind' copied to.
          This is a comma-separated list, which may contain address
          and phrase portions.

    - body
          The body of the email being sent.

    - cc
          The email-address(es) the mail is being copied to. This is a
          comma-separated list, which may contain address and phrase
          portions.

    - charset
          The character set of email being sent

    - from
          The email-address the mail is sent from. May contain address
          and phrase.

    - headers
          A vertical-bar-separated list of headers which should be
          added to the message. Each individual header is specified as
          `header=value' (see example below).

    - host
          The mail server

    - port
          The mail server port

    = subject
          The subject of the email being sent.

    - to
          The email-address(es) the mail is being sent to. This is a
          comma-separated list, which may contain address and phrase
          portions.

    # Example playbook sending mail to root
    - local_action: mail msg='System {{ ansible_hostname }} has been successfully provisioned.'
    
    # Send e-mail to a bunch of users, attaching files
    - local_action: mail
                    host='127.0.0.1'
                    port=2025
                    subject="Ansible-report"
                    body="Hello, this is an e-mail. I hope you like it ;-)"
                    from="jane@example.net (Jane Jolie)"
                    to="John Doe <j.d@example.org>, Suzie Something <sue@example.com>"
                    cc="Charlie Root <root@localhost>"
                    attach="/etc/group /tmp/pavatar2.png"
                    headers=Reply-To=john@example.com|X-Special="Something or other"
                    charset=utf8

> MODPROBE
=======================

::

    Add or remove kernel modules.

    Options (= is mandatory):

    = name
          Name of kernel module to manage.

    - state
          Whether the module should be present or absent. (Choices:
          present, absent)

    # Add the 802.1q module
    - modprobe: name=8021q state=present

> MONGODB_USER
=======================

::

    Adds or removes a user from a MongoDB database.

    Options (= is mandatory):

    = database
          The name of the database to add/remove the user from

    - login_host
          The host running the database

    - login_password
          The password used to authenticate with

    - login_port
          The port to connect to

    - login_user
          The username used to authenticate with

    - password
          The password to use for the user

    - roles
          The database user roles valid values are one or more of the
          following: read, 'readWrite', 'dbAdmin', 'userAdmin',
          'clusterAdmin', 'readAnyDatabase', 'readWriteAnyDatabase',
          'userAdminAnyDatabase', 'dbAdminAnyDatabase'This param
          requires mongodb 2.4+ and pymongo 2.5+

    - state
          The database user state (Choices: present, absent)

    = user
          The name of the user to add or remove

    Notes:    Requires the pymongo Python package on the remote host, version
          2.4.2+. This can be installed using pip or the OS package
          manager. @see
          http://api.mongodb.org/python/current/installation.html

    Requirements:    pymongo

    # Create 'burgers' database user with name 'bob' and password '12345'.
    - mongodb_user: database=burgers name=bob password=12345 state=present
    
    # Delete 'burgers' database user with name 'bob'.
    - mongodb_user: database=burgers name=bob state=absent
    
    # Define more users with various specific roles (if not defined, no roles is assigned, and the user will be added via pre mongo 2.2 style)
    - mongodb_user: database=burgers name=ben password=12345 roles='read' state=present
    - mongodb_user: database=burgers name=jim password=12345 roles='readWrite,dbAdmin,userAdmin' state=present
    - mongodb_user: database=burgers name=joe password=12345 roles='readWriteAnyDatabase' state=present

> MONIT
=======================

::

    Manage the state of a program monitored via `Monit'

    Options (= is mandatory):

    = name
          The name of the `monit' program/process to manage

    = state
          The state of service (Choices: present, started, stopped,
          restarted, monitored, unmonitored, reloaded)

    # Manage the state of program "httpd" to be in "started" state.
    - monit: name=httpd state=started

> MOUNT
=======================

::

    This module controls active and configured mount points in
    `/etc/fstab'.

    Options (= is mandatory):

    - dump
          dump (see fstab(8))

    = fstype
          file-system type

    = name
          path to the mount point, eg: `/mnt/files'

    - opts
          mount options (see fstab(8))

    - passno
          passno (see fstab(8))

    = src
          device to be mounted on `name'.

    = state
          If `mounted' or `unmounted', the device will be actively
          mounted or unmounted as well as just configured in `fstab'.
          `absent' and `present' only deal with `fstab'. `mounted'
          will also automatically create the mount point directory if
          it doesn't exist. If `absent' changes anything, it will
          remove the mount point directory. (Choices: present, absent,
          mounted, unmounted)

    # Mount DVD read-only
    - mount: name=/mnt/dvd src=/dev/sr0 fstype=iso9660 opts=ro state=present
    
    # Mount up device by label
    - mount: name=/srv/disk src='LABEL=SOME_LABEL' state=present
    
    # Mount up device by UUID
    - mount: name=/home src='UUID=b3e48f45-f933-4c8e-a700-22a159ec9077' opts=noatime state=present

> MQTT
=======================

::

    Publish a message on an MQTT topic.

    Options (= is mandatory):

    - client_id
          MQTT client identifier

    - password
          Password for `username' to authenticate against the broker.

    = payload
          Payload. The special string `"None"' may be used to send a
          NULL (i.e. empty) payload which is useful to simply notify
          with the `topic' or to clear previously retained messages.

    - port
          MQTT broker port number

    - qos
          QoS (Quality of Service) (Choices: 0, 1, 2)

    - retain
          Setting this flag causes the broker to retain (i.e. keep)
          the message so that applications that subsequently subscribe
          to the topic can received the last retained message
          immediately.

    - server
          MQTT broker address/name

    = topic
          MQTT topic name

    - username
          Username to authenticate against the broker.

    Notes:    This module requires a connection to an MQTT broker such as
          Mosquitto http://mosquitto.org and the `mosquitto' Python
          module (http://mosquitto.org/python).

    Requirements:    mosquitto

    - local_action: mqtt
                  topic=service/ansible/{{ ansible_hostname }}
                  payload="Hello at {{ ansible_date_time.iso8601 }}"
                  qos=0
                  retain=false
                  client_id=ans001

> MYSQL_DB
=======================

::

    Add or remove MySQL databases from a remote host.

    Options (= is mandatory):

    - collation
          Collation mode

    - encoding
          Encoding mode

    - login_host
          Host running the database

    - login_password
          The password used to authenticate with

    - login_port
          Port of the MySQL server

    - login_unix_socket
          The path to a Unix domain socket for local connections

    - login_user
          The username used to authenticate with

    = name
          name of the database to add or remove

    - state
          The database state (Choices: present, absent, dump, import)

    - target
          Location, on the remote host, of the dump file to read from
          or write to. Uncompressed SQL files (`.sql') as well as
          bzip2 (`.bz2') and gzip (`.gz') compressed files are
          supported.

    Notes:    Requires the MySQLdb Python package on the remote host. For
          Ubuntu, this is as easy as apt-get install python-mysqldb.
          (See [apt].)Both `login_password' and `login_user' are
          required when you are passing credentials. If none are
          present, the module will attempt to read the credentials
          from `~/.my.cnf', and finally fall back to using the MySQL
          default login of `root' with no password.

    Requirements:    ConfigParser

    # Create a new database with name 'bobdata'
    - mysql_db: name=bobdata state=present
    
    # Copy database dump file to remote host and restore it to database 'my_db'
    - copy: src=dump.sql.bz2 dest=/tmp
    - mysql_db: name=my_db state=import target=/tmp/dump.sql.bz2

> MYSQL_REPLICATION
=======================

::

    Manages MySQL server replication, slave, master status get and
    change master host.

    Options (= is mandatory):

    - login_host
          mysql host to connect

    - login_password
          password to connect mysql host, if defined login_user also
          needed.

    - login_unix_socket
          unix socket to connect mysql server

    - login_user
          username to connect mysql host, if defined login_password
          also needed.

    - master_connect_retry
          same as mysql variable

    - master_host
          same as mysql variable

    - master_log_file
          same as mysql variable

    - master_log_pos
          same as mysql variable

    - master_password
          same as mysql variable

    - master_port
          same as mysql variable

    - master_ssl
          same as mysql variable

    - master_ssl_ca
          same as mysql variable

    - master_ssl_capath
          same as mysql variable

    - master_ssl_cert
          same as mysql variable

    - master_ssl_cipher
          same as mysql variable

    - master_ssl_key
          same as mysql variable

    - master_user
          same as mysql variable

    - mode
          module operating mode. Could be getslave (SHOW SLAVE
          STATUS), getmaster (SHOW MASTER STATUS), changemaster
          (CHANGE MASTER TO), startslave (START SLAVE), stopslave
          (STOP SLAVE) (Choices: getslave, getmaster, changemaster,
          stopslave, startslave)

    - relay_log_file
          same as mysql variable

    - relay_log_pos
          same as mysql variable

    # Stop mysql slave thread
    - mysql_replication: mode=stopslave
    
    # Get master binlog file name and binlog position
    - mysql_replication: mode=getmaster
    
    # Change master to master server 192.168.1.1 and use binary log 'mysql-bin.000009' with position 4578
    - mysql_replication: mode=changemaster master_host=192.168.1.1 master_log_file=mysql-bin.000009 master_log_pos=4578

> MYSQL_USER
=======================

::

    Adds or removes a user from a MySQL database.

    Options (= is mandatory):

    - append_privs
          Append the privileges defined by priv to the existing ones
          for this user instead of overwriting existing ones.
          (Choices: yes, no)

    - check_implicit_admin
          Check if mysql allows login as root/nopassword before trying
          supplied credentials.

    - host
          the 'host' part of the MySQL username

    - login_host
          Host running the database

    - login_password
          The password used to authenticate with

    - login_port
          Port of the MySQL server

    - login_unix_socket
          The path to a Unix domain socket for local connections

    - login_user
          The username used to authenticate with

    = name
          name of the user (role) to add or remove

    - password
          set the user's password

    - priv
          MySQL privileges string in the format:
          `db.table:priv1,priv2'

    - state
          Whether the user should exist.  When `absent', removes the
          user. (Choices: present, absent)

    Notes:    Requires the MySQLdb Python package on the remote host. For
          Ubuntu, this is as easy as apt-get install python-
          mysqldb.Both `login_password' and `login_username' are
          required when you are passing credentials. If none are
          present, the module will attempt to read the credentials
          from `~/.my.cnf', and finally fall back to using the MySQL
          default login of 'root' with no password.MySQL server
          installs with default login_user of 'root' and no password.
          To secure this user as part of an idempotent playbook, you
          must create at least two tasks: the first must change the
          root user's password, without providing any
          login_user/login_password details. The second must drop a
          ~/.my.cnf file containing the new root credentials.
          Subsequent runs of the playbook will then succeed by reading
          the new credentials from the file.

    Requirements:    ConfigParser, MySQLdb

    # Create database user with name 'bob' and password '12345' with all database privileges
    - mysql_user: name=bob password=12345 priv=*.*:ALL state=present
    
    # Creates database user 'bob' and password '12345' with all database privileges and 'WITH GRANT OPTION'
    - mysql_user: name=bob password=12345 priv=*.*:ALL,GRANT state=present
    
    # Ensure no user named 'sally' exists, also passing in the auth credentials.
    - mysql_user: login_user=root login_password=123456 name=sally state=absent
    
    # Example privileges string format
    mydb.*:INSERT,UPDATE/anotherdb.*:SELECT/yetanotherdb.*:ALL
    
    # Example using login_unix_socket to connect to server
    - mysql_user: name=root password=abc123 login_unix_socket=/var/run/mysqld/mysqld.sock
    
    # Example .my.cnf file for setting the root password
    # Note: don't use quotes around the password, because the mysql_user module
    # will include them in the password but the mysql client will not
    
    [client]
    user=root
    password=n<_665{vS43y

> MYSQL_VARIABLES
=======================

::

    Query / Set MySQL variables

    Options (= is mandatory):

    - login_host
          mysql host to connect

    - login_password
          password to connect mysql host, if defined login_user also
          needed.

    - login_unix_socket
          unix socket to connect mysql server

    - login_user
          username to connect mysql host, if defined login_password
          also needed.

    - value
          If set, then sets variable value to this

    = variable
          Variable name to operate

    # Check for sync_binary_log setting
    - mysql_variables: variable=sync_binary_log
    
    # Set read_only variable to 1
    - mysql_variables: variable=read_only value=1

> NAGIOS
=======================

::

    The [nagios] module has two basic functions: scheduling downtime
    and toggling alerts for services or hosts.All actions require the
    `host' parameter to be given explicitly. In playbooks you can use
    the `{{inventory_hostname}}' variable to refer to the host the
    playbook is currently running on.You can specify multiple services
    at once by separating them with commas, .e.g.,
    `services=httpd,nfs,puppet'.When specifying what service to handle
    there is a special service value, `host', which will handle
    alerts/downtime for the `host itself', e.g., `service=host'. This
    keyword may not be given with other services at the same time.
    `Setting alerts/downtime for a host does not affect
    alerts/downtime for any of the services running on it.' To
    schedule downtime for all services on particular host use keyword
    "all", e.g., `service=all'.When using the [nagios] module you will
    need to specify your Nagios server using the `delegate_to'
    parameter.

    Options (= is mandatory):

    = action
          Action to take. (Choices: downtime, enable_alerts,
          disable_alerts, silence, unsilence, silence_nagios,
          unsilence_nagios, command)

    - author
          Author to leave downtime comments as. Only usable with the
          `downtime' action.

    - cmdfile
          Path to the nagios `command file' (FIFO pipe). Only required
          if auto-detection fails.

    = command
          The raw command to send to nagios, which should not include
          the submitted time header or the line-feed *Required* option
          when using the `command' action.

    - host
          Host to operate on in Nagios.

    - minutes
          Minutes to schedule downtime for.Only usable with the
          `downtime' action.

    = services
          What to manage downtime/alerts for. Separate multiple
          services with commas. `service' is an alias for `services'.
          *Required* option when using the `downtime',
          `enable_alerts', and `disable_alerts' actions.

    Requirements:    Nagios

    # set 30 minutes of apache downtime
    - nagios: action=downtime minutes=30 service=httpd host={{ inventory_hostname }}
    
    # schedule an hour of HOST downtime
    - nagios: action=downtime minutes=60 service=host host={{ inventory_hostname }}
    
    # schedule downtime for ALL services on HOST
    - nagios: action=downtime minutes=45 service=all host={{ inventory_hostname }}
    
    # schedule downtime for a few services
    - nagios: action=downtime services=frob,foobar,qeuz host={{ inventory_hostname }}
    
    # enable SMART disk alerts
    - nagios: action=enable_alerts service=smart host={{ inventory_hostname }}
    
    # "two services at once: disable httpd and nfs alerts"
    - nagios: action=disable_alerts service=httpd,nfs host={{ inventory_hostname }}
    
    # disable HOST alerts
    - nagios: action=disable_alerts service=host host={{ inventory_hostname }}
    
    # silence ALL alerts
    - nagios: action=silence host={{ inventory_hostname }}
    
    # unsilence all alerts
    - nagios: action=unsilence host={{ inventory_hostname }}
    
    # SHUT UP NAGIOS
    - nagios: action=silence_nagios
    
    # ANNOY ME NAGIOS
    - nagios: action=unsilence_nagios
    
    # command something
    - nagios: action=command command='DISABLE_FAILURE_PREDICTION'

> NETSCALER
=======================

::

    Manages Citrix NetScaler server and service entities.

    Options (= is mandatory):

    - action
          the action you want to perform on the entity (Choices:
          enable, disable)

    = name
          name of the entity

    = nsc_host
          hostname or ip of your netscaler

    - nsc_protocol
          protocol used to access netscaler

    = password
          password

    - type
          type of the entity (Choices: server, service)

    = user
          username

    - validate_certs
          If `no', SSL certificates for the target url will not be
          validated. This should only be used on personally controlled
          sites using self-signed certificates. (Choices: yes, no)

    Requirements:    urllib, urllib2

    # Disable the server
    ansible host -m netscaler -a "nsc_host=nsc.example.com user=apiuser password=apipass"
    
    # Enable the server
    ansible host -m netscaler -a "nsc_host=nsc.example.com user=apiuser password=apipass action=enable"
    
    # Disable the service local:8080
    ansible host -m netscaler -a "nsc_host=nsc.example.com user=apiuser password=apipass name=local:8080 type=service action=disable"

> NEWRELIC_DEPLOYMENT
=======================

::

    Notify newrelic about app deployments (see http://newrelic.github.
    io/newrelic_api/NewRelicApi/Deployment.html)

    Options (= is mandatory):

    - app_name
          (one of app_name or application_id are required) The value
          of app_name in the newrelic.yml file used by the application

    - application_id
          (one of app_name or application_id are required) The
          application id, found in the URL when viewing the
          application in RPM

    - appname
          Name of the application

    - changelog
          A list of changes for this deployment

    - description
          Text annotation for the deployment - notes for you

    - environment
          The environment for this deployment

    - revision
          A revision number (e.g., git commit SHA)

    = token
          API token.

    - user
          The name of the user/process that triggered this deployment

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Requirements:    urllib, urllib2

    - newrelic_deployment: token=AAAAAA
                           app_name=myapp
                           user='ansible deployment'
                           revision=1.0

> NOVA_COMPUTE
=======================

::

    Create or Remove virtual machines from Openstack.

    Options (= is mandatory):

    - auth_url
          The keystone url for authentication

    - flavor_id
          The id of the flavor in which the new VM has to be created

    = image_id
          The id of the image that has to be cloned

    - key_name
          The key pair name to be used when creating a VM

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    - meta
          A list of key value pairs that should be provided as a
          metadata to the new VM

    = name
          Name that has to be given to the instance

    - nics
          A list of network id's to which the VM's interface should be
          attached

    - region_name
          Name of the region

    - security_groups
          The name of the security group to which the VM should be
          added

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - wait
          If the module should wait for the VM to be created.

    - wait_for
          The amount of time the module should wait for the VM to get
          into active state

    Requirements:    novaclient

    # Creates a new VM and attaches to a network and passes metadata to the instance
    - nova_compute:
           state: present
           login_username: admin
           login_password: admin
           login_tenant_name: admin
           name: vm1
           image_id: 4f905f38-e52a-43d2-b6ec-754a13ffb529
           key_name: ansible_key
           wait_for: 200
           flavor_id: 4
           nics:
             - net-id: 34605f38-e52a-25d2-b6ec-754a13ffb723
           meta:
             hostname: test1
             group: uge_master

> NOVA_KEYPAIR
=======================

::

    Add or Remove key pair from nova .

    Options (= is mandatory):

    - auth_url
          The keystone url for authentication

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    = name
          Name that has to be given to the key pair

    - public_key
          The public key that would be uploaded to nova and injected
          to vm's upon creation

    - region_name
          Name of the region

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    Requirements:    novaclient

    # Creates a key pair with the running users public key
    - nova_keypair: state=present login_username=admin
                    login_password=admin login_tenant_name=admin name=ansible_key
                    public_key={{ lookup('file','~/.ssh/id_rsa.pub') }}
    
    # Creates a new key pair and the private key returned after the run.
    - nova_keypair: state=present login_username=admin login_password=admin
                    login_tenant_name=admin name=ansible_key

> NPM
=======================

::

    Manage node.js packages with Node Package Manager (npm)

    Options (= is mandatory):

    - executable
          The executable location for npm.This is useful if you are
          using a version manager, such as nvm

    - global
          Install the node.js library globally (Choices: yes, no)

    - name
          The name of a node.js library to install

    - path
          The base path where to install the node.js libraries

    - production
          Install dependencies in production mode, excluding
          devDependencies (Choices: yes, no)

    - state
          The state of the node.js library (Choices: present, absent,
          latest)

    - version
          The version to be installed

    description: Install "coffee-script" node.js package.
    - npm: name=coffee-script path=/app/location
    
    description: Install "coffee-script" node.js package on version 1.6.1.
    - npm: name=coffee-script version=1.6.1 path=/app/location
    
    description: Install "coffee-script" node.js package globally.
    - npm: name=coffee-script global=yes
    
    description: Remove the globally package "coffee-script".
    - npm: name=coffee-script global=yes state=absent
    
    description: Install packages based on package.json.
    - npm: path=/app/location
    
    description: Update packages based on package.json to their latest version.
    - npm: path=/app/location state=latest
    
    description: Install packages based on package.json using the npm installed with nvm v0.10.1.
    - npm: path=/app/location executable=/opt/nvm/v0.10.1/bin/npm state=present

> OHAI
=======================

::

    Similar to the [facter] module, this runs the `Ohai' discovery
    program (http://wiki.opscode.com/display/chef/Ohai) on the remote
    host and returns JSON inventory data. `Ohai' data is a bit more
    verbose and nested than `facter'.

    Requirements:    ohai

    # Retrieve (ohai) data from all Web servers and store in one-file per host
    ansible webservers -m ohai --tree=/tmp/ohaidata

> OPEN_ISCSI
=======================

::

    Discover targets on given portal, (dis)connect targets, mark
    targets to manually or auto start, return device nodes of
    connected targets.

    Options (= is mandatory):

    - auto_node_startup
          whether the target node should be automatically connected at
          startup (Choices: True, False)

    - discover
          whether the list of target nodes on the portal should be
          (re)discovered and added to the persistent iscsi database.
          Keep in mind that iscsiadm discovery resets configurtion,
          like node.startup to manual, hence combined with
          auto_node_startup=yes will allways return a changed state.
          (Choices: True, False)

    - login
          whether the target node should be connected (Choices: True,
          False)

    - node_auth
          discovery.sendtargets.auth.authmethod

    - node_pass
          discovery.sendtargets.auth.password

    - node_user
          discovery.sendtargets.auth.username

    - port
          the port on which the iscsi target process listens

    - portal
          the ip address of the iscsi target

    - show_nodes
          whether the list of nodes in the persistent iscsi database
          should be returned by the module (Choices: True, False)

    - target
          the iscsi target name

    Requirements:    open_iscsi library and tools (iscsiadm)

    Examples:

    open_iscsi: show_nodes=yes discover=yes portal=10.1.2.3


    open_iscsi: portal={{iscsi_target}} login=yes discover=yes


    open_iscsi: login=yes target=iqn.1986-03.com.sun:02:f8c1f9e0-c3ec-ec84-c9c9-8bfb0cd5de3d


    open_iscsi: login=no target=iqn.1986-03.com.sun:02:f8c1f9e0-c3ec-ec84-c9c9-8bfb0cd5de3d"



> OPENBSD_PKG
=======================

::

    Manage packages on OpenBSD using the pkg tools.

    Options (= is mandatory):

    = name
          Name of the package.

    = state
          `present' will make sure the package is installed. `latest'
          will make sure the latest version of the package is
          installed. `absent' will make sure the specified package is
          not installed. (Choices: present, latest, absent)

    # Make sure nmap is installed
    - openbsd_pkg: name=nmap state=present
    
    # Make sure nmap is the latest version
    - openbsd_pkg: name=nmap state=latest
    
    # Make sure nmap is not installed
    - openbsd_pkg: name=nmap state=absent

> OPENVSWITCH_BRIDGE
=======================

::

    Manage Open vSwitch bridges

    Options (= is mandatory):

    = bridge
          Name of bridge to manage

    - state
          Whether the bridge should exist (Choices: present, absent)

    - timeout
          How long to wait for ovs-vswitchd to respond

    Requirements:    ovs-vsctl

    # Create a bridge named br-int
    - openvswitch_bridge: bridge=br-int state=present

> OPENVSWITCH_PORT
=======================

::

    Manage Open vSwitch ports

    Options (= is mandatory):

    = bridge
          Name of bridge to manage

    = port
          Name of port to manage on the bridge

    - state
          Whether the port should exist (Choices: present, absent)

    - timeout
          How long to wait for ovs-vswitchd to respond

    Requirements:    ovs-vsctl

    # Creates port eth2 on bridge br-ex
    - openvswitch_port: bridge=br-ex port=eth2 state=present

> OPKG
=======================

::

    Manages OpenWrt packages

    Options (= is mandatory):

    = name
          name of package to install/remove

    - state
          state of the package (Choices: present, absent)

    - update_cache
          update the package db first (Choices: yes, no)

    - opkg: name=foo state=present
    - opkg: name=foo state=present update_cache=yes
    - opkg: name=foo state=absent
    - opkg: name=foo,bar state=absent

> OSX_SAY
=======================

::

    makes an OS computer speak!  Amuse your friends, annoy your
    coworkers!

    Options (= is mandatory):

    = msg
          What to say

    - voice
          What voice to use

    Notes:    If you like this module, you may also be interested in the osx_say
          callback in the plugins/ directory of the source checkout.

    Requirements:    say

    - local_action: osx_say msg="{{inventory_hostname}} is all done" voice=Zarvox

> OVIRT
=======================

::

    allows you to create new instances, either from scratch or an
    image, in addition to deleting or stopping instances on the
    oVirt/RHEV platform

    Options (= is mandatory):

    - disk_alloc
          define if disk is thin or preallocated (Choices: thin,
          preallocated)

    - disk_int
          interface type of the disk (Choices: virtio, ide)

    - image
          template to use for the instance

    - instance_cores
          define the instance's number of cores

    - instance_cpus
          the instance's number of cpu's

    - instance_disksize
          size of the instance's disk in GB

    - instance_mem
          the instance's amount of memory in MB

    = instance_name
          the name of the instance to use

    - instance_network
          the logical network the machine should belong to

    - instance_nic
          name of the network interface in oVirt/RHEV

    - instance_os
          type of Operating System

    - instance_type
          define if the instance is a server or desktop (Choices:
          server, desktop)

    = password
          password of the user to authenticate with

    - region
          the oVirt/RHEV datacenter where you want to deploy to

    - resource_type
          whether you want to deploy an image or create an instance
          from scratch. (Choices: new, template)

    - sdomain
          the Storage Domain where you want to create the instance's
          disk on.

    - state
          create, terminate or remove instances (Choices: present,
          absent, shutdown, started, restarted)

    = url
          the url of the oVirt instance

    = user
          the user to authenticate with

    - zone
          deploy the image to this oVirt cluster

    Requirements:    ovirt-engine-sdk

    # Basic example provisioning from image.
    
    action: ovirt >
        user=admin@internal 
        url=https://ovirt.example.com 
        instance_name=ansiblevm04 
        password=secret 
        image=centos_64 
        zone=cluster01 
        resource_type=template"
    
    # Full example to create new instance from scratch
    action: ovirt > 
        instance_name=testansible 
        resource_type=new 
        instance_type=server 
        user=admin@internal 
        password=secret 
        url=https://ovirt.example.com 
        instance_disksize=10 
        zone=cluster01 
        region=datacenter1 
        instance_cpus=1 
        instance_nic=nic1 
        instance_network=rhevm 
        instance_mem=1000 
        disk_alloc=thin 
        sdomain=FIBER01 
        instance_cores=1 
        instance_os=rhel_6x64 
        disk_int=virtio"
    
    # stopping an instance
    action: ovirt >
        instance_name=testansible
        state=stopped
        user=admin@internal
        password=secret
        url=https://ovirt.example.com
    
    # starting an instance
    action: ovirt >
        instance_name=testansible 
        state=started 
        user=admin@internal 
        password=secret 
        url=https://ovirt.example.com
    
    

> PACMAN
=======================

::

    Manages Archlinux packages

    Options (= is mandatory):

    = name
          name of package to install, upgrade or remove.

    - recurse
          remove all not explicitly installed dependencies not
          required by other packages of the package to remove
          (Choices: yes, no)

    - state
          desired state of the package. (Choices: installed, absent)

    - update_cache
          update the package database first (pacman -Syy). (Choices:
          yes, no)

    # Install package foo
    - pacman: name=foo state=installed
    
    # Remove package foo
    - pacman: name=foo state=absent
    
    # Remove packages foo and bar 
    - pacman: name=foo,bar state=absent
    
    # Recursively remove package baz
    - pacman: name=baz state=absent recurse=yes
    
    # Update the package database (pacman -Syy) and install bar (bar will be the updated if a newer version exists) 
    - pacman: name=bar, state=installed, update_cache=yes

> PAGERDUTY
=======================

::

    This module will let you create PagerDuty maintenance windows

    Options (= is mandatory):

    - desc
          Short description of maintenance window. (Choices: )

    - hours
          Length of maintenance window in hours. (Choices: )

    = name
          PagerDuty unique subdomain. (Choices: )

    = passwd
          PagerDuty user password. (Choices: )

    - service
          PagerDuty service ID. (Choices: )

    = state
          Create a maintenance window or get a list of ongoing
          windows. (Choices: running, started, ongoing)

    = user
          PagerDuty user ID. (Choices: )

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    Notes:    This module does not yet have support to end maintenance windows.

    Requirements:    PagerDuty API access

    # List ongoing maintenance windows.
    - pagerduty: name=companyabc user=example@example.com passwd=password123 state=ongoing
    
    # Create a 1 hour maintenance window for service FOO123.
    - pagerduty: name=companyabc
                 user=example@example.com
                 passwd=password123
                 state=running
                 service=FOO123
    
    # Create a 4 hour maintenance window for service FOO123 with the description "deployment".
    - pagerduty: name=companyabc
                 user=example@example.com
                 passwd=password123
                 state=running
                 service=FOO123
                 hours=4
                 desc=deployment

> PAUSE
=======================

::

    Pauses playbook execution for a set amount of time, or until a
    prompt is acknowledged. All parameters are optional. The default
    behavior is to pause with a prompt.You can use `ctrl+c' if you
    wish to advance a pause earlier than it is set to expire or if you
    need to abort a playbook run entirely. To continue early: press
    `ctrl+c' and then `c'. To abort a playbook: press `ctrl+c' and
    then `a'.The pause module integrates into async/parallelized
    playbooks without any special considerations (see also: Rolling
    Updates). When using pauses with the `serial' playbook parameter
    (as in rolling updates) you are only prompted once for the current
    group of hosts.

    Options (= is mandatory):

    - minutes
          Number of minutes to pause for.

    - prompt
          Optional text to use for the prompt message.

    - seconds
          Number of seconds to pause for.

    # Pause for 5 minutes to build app cache.
    - pause: minutes=5
    
    # Pause until you can verify updates to an application were successful.
    - pause:
    
    # A helpful reminder of what to look out for post-update.
    - pause: prompt="Make sure org.foo.FooOverload exception is not present"

> PING
=======================

::

    A trivial test module, this module always returns `pong' on
    successful contact. It does not make sense in playbooks, but it is
    useful from `/usr/bin/ansible'

    # Test 'webservers' status
    ansible webservers -m ping

> PINGDOM
=======================

::

    This module will let you pause/unpause Pingdom alerts

    Options (= is mandatory):

    = checkid
          Pingdom ID of the check. (Choices: )

    = key
          Pingdom API key. (Choices: )

    = passwd
          Pingdom user password. (Choices: )

    = state
          Define whether or not the check should be running or paused.
          (Choices: running, paused)

    = uid
          Pingdom user ID. (Choices: )

    Notes:    This module does not yet have support to add/remove checks.

    Requirements:    This pingdom python library: https://github.com/mbabineau/pingdom-
          python

    # Pause the check with the ID of 12345.
    - pingdom: uid=example@example.com
               passwd=password123
               key=apipassword123
               checkid=12345
               state=paused
    
    # Unpause the check with the ID of 12345.
    - pingdom: uid=example@example.com
               passwd=password123
               key=apipassword123
               checkid=12345
               state=running

> PIP
=======================

::

    Manage Python library dependencies. To use this module, one of the
    following keys is required: `name' or `requirements'.

    Options (= is mandatory):

    - chdir
          cd into this directory before running the command

    - executable
          The explicit executable or a pathname to the executable to
          be used to run pip for a specific version of Python
          installed in the system. For example `pip-3.3', if there are
          both Python 2.7 and 3.3 installations in the system and you
          want to run pip for the Python 3.3 installation.

    - extra_args
          Extra arguments passed to pip.

    - name
          The name of a Python library to install or the url of the
          remote package.

    - requirements
          The path to a pip requirements file

    - state
          The state of module (Choices: present, absent, latest)

    - version
          The version number to install of the Python library
          specified in the `name' parameter

    - virtualenv
          An optional path to a `virtualenv' directory to install into

    - virtualenv_command
          The command or a pathname to the command to create the
          virtual environment with. For example `pyvenv',
          `virtualenv', `virtualenv2', `~/bin/virtualenv',
          `/usr/local/bin/virtualenv'.

    - virtualenv_site_packages
          Whether the virtual environment will inherit packages from
          the global site-packages directory.  Note that if this
          setting is changed on an already existing virtual
          environment it will not have any effect, the environment
          must be deleted and newly created. (Choices: yes, no)

    Notes:    Please note that virtualenv (http://www.virtualenv.org/) must be
          installed on the remote host if the virtualenv parameter is
          specified.

    Requirements:    virtualenv, pip

    # Install (Bottle) python package.
    - pip: name=bottle
    
    # Install (Bottle) python package on version 0.11.
    - pip: name=bottle version=0.11
    
    # Install (MyApp) using one of the remote protocols (bzr+,hg+,git+,svn+). You do not have to supply '-e' option in extra_args.
    - pip: name='svn+http://myrepo/svn/MyApp#egg=MyApp'
    
    # Install (Bottle) into the specified (virtualenv), inheriting none of the globally installed modules
    - pip: name=bottle virtualenv=/my_app/venv
    
    # Install (Bottle) into the specified (virtualenv), inheriting globally installed modules
    - pip: name=bottle virtualenv=/my_app/venv virtualenv_site_packages=yes
    
    # Install (Bottle) into the specified (virtualenv), using Python 2.7
    - pip: name=bottle virtualenv=/my_app/venv virtualenv_command=virtualenv-2.7
    
    # Install specified python requirements.
    - pip: requirements=/my_app/requirements.txt
    
    # Install specified python requirements in indicated (virtualenv).
    - pip: requirements=/my_app/requirements.txt virtualenv=/my_app/venv
    
    # Install specified python requirements and custom Index URL.
    - pip: requirements=/my_app/requirements.txt extra_args='-i https://example.com/pypi/simple'
    
    # Install (Bottle) for Python 3.3 specifically,using the 'pip-3.3' executable.
    - pip: name=bottle executable=pip-3.3

> PKGIN
=======================

::

    Manages SmartOS packages

    Options (= is mandatory):

    = name
          name of package to install/remove

    - state
          state of the package (Choices: present, absent)

    # install package foo"
    - pkgin: name=foo state=present
    
    # remove package foo
    - pkgin: name=foo state=absent
    
    # remove packages foo and bar
    - pkgin: name=foo,bar state=absent

> PKGNG
=======================

::

    Manage binary packages for FreeBSD using 'pkgng' which is
    available in versions after 9.0.

    Options (= is mandatory):

    - cached
          use local package base or try to fetch an updated one
          (Choices: yes, no)

    = name
          name of package to install/remove

    - pkgsite
          specify packagesite to use for downloading packages, if not
          specified, use settings from /usr/local/etc/pkg.conf

    - state
          state of the package (Choices: present, absent)

    Notes:    When using pkgsite, be careful that already in cache packages
          won't be downloaded again.

    # Install package foo
    - pkgng: name=foo state=present
    
    # Remove packages foo and bar 
    - pkgng: name=foo,bar state=absent

> PKGUTIL
=======================

::

    Manages CSW packages (SVR4 format) on Solaris 10 and 11.These were
    the native packages on Solaris <= 10 and are available as a legacy
    feature in Solaris 11.Pkgutil is an advanced packaging system,
    which resolves dependency on installation. It is designed for CSW
    packages.

    Options (= is mandatory):

    = name
          Package name, e.g. (`CSWnrpe')

    - site
          Specifies the repository path to install the package
          from.Its global definition is done in
          `/etc/opt/csw/pkgutil.conf'.

    = state
          Whether to install (`present'), or remove (`absent') a
          package.The upgrade (`latest') operation will update/install
          the package to the latest version available.Note: The module
          has a limitation that (`latest') only works for one package,
          not lists of them. (Choices: present, absent, latest)

    # Install a package
    pkgutil: name=CSWcommon state=present
    
    # Install a package from a specific repository
    pkgutil: name=CSWnrpe site='ftp://myinternal.repo/opencsw/kiel state=latest'

> PORTINSTALL
=======================

::

    Manage packages for FreeBSD using 'portinstall'.

    Options (= is mandatory):

    = name
          name of package to install/remove

    - state
          state of the package (Choices: present, absent)

    - use_packages
          use packages instead of ports whenever available (Choices:
          yes, no)

    # Install package foo
    - portinstall: name=foo state=present
    
    # Install package security/cyrus-sasl2-saslauthd
    - portinstall: name=security/cyrus-sasl2-saslauthd state=present
    
    # Remove packages foo and bar
    - portinstall: name=foo,bar state=absent

> POSTGRESQL_DB
=======================

::

    Add or remove PostgreSQL databases from a remote host.

    Options (= is mandatory):

    - encoding
          Encoding of the database

    - lc_collate
          Collation order (LC_COLLATE) to use in the database. Must
          match collation order of template database unless
          `template0' is used as template.

    - lc_ctype
          Character classification (LC_CTYPE) to use in the database
          (e.g. lower, upper, ...) Must match LC_CTYPE of template
          database unless `template0' is used as template.

    - login_host
          Host running the database

    - login_password
          The password used to authenticate with

    - login_user
          The username used to authenticate with

    = name
          name of the database to add or remove

    - owner
          Name of the role to set as owner of the database

    - port
          Database port to connect to.

    - state
          The database state (Choices: present, absent)

    - template
          Template used to create the database

    Notes:    The default authentication assumes that you are either logging in
          as or sudo'ing to the `postgres' account on the host.This
          module uses `psycopg2', a Python PostgreSQL database
          adapter. You must ensure that psycopg2 is installed on the
          host before using this module. If the remote host is the
          PostgreSQL server (which is the default case), then
          PostgreSQL must also be installed on the remote host. For
          Ubuntu-based systems, install the `postgresql', `libpq-dev',
          and `python-psycopg2' packages on the remote host before
          using this module.

    Requirements:    psycopg2

    # Create a new database with name "acme"
    - postgresql_db: name=acme
    
    # Create a new database with name "acme" and specific encoding and locale
    # settings. If a template different from "template0" is specified, encoding
    # and locale settings must match those of the template.
    - postgresql_db: name=acme
                     encoding='UTF-8'
                     lc_collate='de_DE.UTF-8'
                     lc_ctype='de_DE.UTF-8'
                     template='template0'

> POSTGRESQL_PRIVS
=======================

::

    Grant or revoke privileges on PostgreSQL database objects.This
    module is basically a wrapper around most of the functionality of
    PostgreSQL's GRANT and REVOKE statements with detection of changes
    (GRANT/REVOKE `privs' ON `type' `objs' TO/FROM `roles')

    Options (= is mandatory):

    = database
          Name of database to connect to.Alias: `db'

    - grant_option
          Whether `role' may grant/revoke the specified
          privileges/group memberships to others.Set to `no' to revoke
          GRANT OPTION, leave unspecified to make no
          changes.`grant_option' only has an effect if `state' is
          `present'.Alias: `admin_option' (Choices: yes, no)

    - host
          Database host address. If unspecified, connect via Unix
          socket.Alias: `login_host'

    - login
          The username to authenticate with.Alias: `login_user'

    - objs
          Comma separated list of database objects to set privileges
          on.If `type' is `table' or `sequence', the special value
          `ALL_IN_SCHEMA' can be provided instead to specify all
          database objects of type `type' in the schema specified via
          `schema'. (This also works with PostgreSQL < 9.0.)If `type'
          is `database', this parameter can be omitted, in which case
          privileges are set for the database specified via
          `database'.If `type' is `function', colons (":") in object
          names will be replaced with commas (needed to specify
          function signatures, see examples)Alias: `obj'

    - password
          The password to authenticate with.Alias: `login_password')

    - port
          Database port to connect to.

    - privs
          Comma separated list of privileges to grant/revoke.Alias:
          `priv'

    = roles
          Comma separated list of role (user/group) names to set
          permissions for.The special value `PUBLIC' can be provided
          instead to set permissions for the implicitly defined PUBLIC
          group.Alias: `role'

    - schema
          Schema that contains the database objects specified via
          `objs'.May only be provided if `type' is `table', `sequence'
          or `function'. Defaults to  `public' in these cases.

    - state
          If `present', the specified privileges are granted, if
          `absent' they are revoked. (Choices: present, absent)

    - type
          Type of database object to set privileges on. (Choices:
          table, sequence, function, database, schema, language,
          tablespace, group)

    Notes:    Default authentication assumes that postgresql_privs is run by the
          `postgres' user on the remote host. (Ansible's `user' or
          `sudo-user').This module requires Python package `psycopg2'
          to be installed on the remote host. In the default case of
          the remote host also being the PostgreSQL server, PostgreSQL
          has to be installed there as well, obviously. For Debian
          /Ubuntu-based systems, install packages `postgresql' and
          `python-psycopg2'.Parameters that accept comma separated
          lists (`privs', `objs', `roles') have singular alias names
          (`priv', `obj', `role').To revoke only `GRANT OPTION' for a
          specific object, set `state' to `present' and `grant_option'
          to `no' (see examples).Note that when revoking privileges
          from a role R, this role  may still have access via
          privileges granted to any role R is a member of including
          `PUBLIC'.Note that when revoking privileges from a role R,
          you do so as the user specified via `login'. If R has been
          granted the same privileges by another user also, R can
          still access database objects via these privileges.When
          revoking privileges, `RESTRICT' is assumed (see PostgreSQL
          docs).

    Requirements:    psycopg2

    # On database "library":
    # GRANT SELECT, INSERT, UPDATE ON TABLE public.books, public.authors 
    # TO librarian, reader WITH GRANT OPTION
    - postgresql_privs: >
        database=library
        state=present
        privs=SELECT,INSERT,UPDATE
        type=table
        objs=books,authors
        schema=public
        roles=librarian,reader
        grant_option=yes
    
    # Same as above leveraging default values:
    - postgresql_privs: >
        db=library
        privs=SELECT,INSERT,UPDATE
        objs=books,authors
        roles=librarian,reader
        grant_option=yes
    
    # REVOKE GRANT OPTION FOR INSERT ON TABLE books FROM reader 
    # Note that role "reader" will be *granted* INSERT privilege itself if this 
    # isn't already the case (since state=present).
    - postgresql_privs: >
        db=library
        state=present
        priv=INSERT
        obj=books
        role=reader
        grant_option=no
    
    # REVOKE INSERT, UPDATE ON ALL TABLES IN SCHEMA public FROM reader
    # "public" is the default schema. This also works for PostgreSQL 8.x.
    - postgresql_privs: >
        db=library
        state=absent
        privs=INSERT,UPDATE
        objs=ALL_IN_SCHEMA
        role=reader
    
    # GRANT ALL PRIVILEGES ON SCHEMA public, math TO librarian
    - postgresql_privs: >
        db=library
        privs=ALL
        type=schema
        objs=public,math
        role=librarian
    
    # GRANT ALL PRIVILEGES ON FUNCTION math.add(int, int) TO librarian, reader
    # Note the separation of arguments with colons.
    - postgresql_privs: >
        db=library
        privs=ALL
        type=function
        obj=add(int:int)
        schema=math
        roles=librarian,reader
    
    # GRANT librarian, reader TO alice, bob WITH ADMIN OPTION
    # Note that group role memberships apply cluster-wide and therefore are not
    # restricted to database "library" here.
    - postgresql_privs: >
        db=library
        type=group
        objs=librarian,reader
        roles=alice,bob
        admin_option=yes
    
    # GRANT ALL PRIVILEGES ON DATABASE library TO librarian
    # Note that here "db=postgres" specifies the database to connect to, not the
    # database to grant privileges on (which is specified via the "objs" param)
    - postgresql_privs: >
        db=postgres
        privs=ALL
        type=database
        obj=library
        role=librarian
    
    # GRANT ALL PRIVILEGES ON DATABASE library TO librarian
    # If objs is omitted for type "database", it defaults to the database 
    # to which the connection is established
    - postgresql_privs: >
        db=library
        privs=ALL
        type=database
        role=librarian

> POSTGRESQL_USER
=======================

::

    Add or remove PostgreSQL users (roles) from a remote host and,
    optionally, grant the users access to an existing database or
    tables.The fundamental function of the module is to create, or
    delete, roles from a PostgreSQL cluster. Privilege assignment, or
    removal, is an optional step, which works on one database at a
    time. This allows for the module to be called several times in the
    same module to modify the permissions on different databases, or
    to grant permissions to already existing users.A user cannot be
    removed until all the privileges have been stripped from the user.
    In such situation, if the module tries to remove the user it will
    fail. To avoid this from happening the fail_on_user option signals
    the module to try to remove the user, but if not possible keep
    going; the module will report if changes happened and separately
    if the user was removed or not.

    Options (= is mandatory):

    - db
          name of database where permissions will be granted

    - encrypted
          denotes if the password is already encrypted. boolean.

    - expires
          sets the user's password expiration.

    - fail_on_user
          if `yes', fail when user can't be removed. Otherwise just
          log and continue (Choices: yes, no)

    - login_host
          Host running PostgreSQL.

    - login_password
          Password used to authenticate with PostgreSQL

    - login_user
          User (role) used to authenticate with PostgreSQL

    = name
          name of the user (role) to add or remove

    - password
          set the user's password, before 1.4 this was required.When
          passing an encrypted password it must be generated with the
          format `'str["md5"] + md5[ password + username ]'',
          resulting in a total of 35 characters.  An easy way to do
          this is: `echo "md5`echo -n "verysecretpasswordJOE" |
          md5`"'.

    - port
          Database port to connect to.

    - priv
          PostgreSQL privileges string in the format:
          `table:priv1,priv2'

    - role_attr_flags
          PostgreSQL role attributes string in the format:
          CREATEDB,CREATEROLE,SUPERUSER (Choices: [NO]SUPERUSER,
          [NO]CREATEROLE, [NO]CREATEUSER, [NO]CREATEDB, [NO]INHERIT,
          [NO]LOGIN, [NO]REPLICATION)

    - state
          The user (role) state (Choices: present, absent)

    Notes:    The default authentication assumes that you are either logging in
          as or sudo'ing to the postgres account on the host.This
          module uses psycopg2, a Python PostgreSQL database adapter.
          You must ensure that psycopg2 is installed on the host
          before using this module. If the remote host is the
          PostgreSQL server (which is the default case), then
          PostgreSQL must also be installed on the remote host. For
          Ubuntu-based systems, install the postgresql, libpq-dev, and
          python-psycopg2 packages on the remote host before using
          this module.If you specify PUBLIC as the user, then the
          privilege changes will apply to all users. You may not
          specify password or role_attr_flags when the PUBLIC user is
          specified.

    Requirements:    psycopg2

    # Create django user and grant access to database and products table
    - postgresql_user: db=acme name=django password=ceec4eif7ya priv=CONNECT/products:ALL
    
    # Create rails user, grant privilege to create other databases and demote rails from super user status
    - postgresql_user: name=rails password=secret role_attr_flags=CREATEDB,NOSUPERUSER
    
    # Remove test user privileges from acme
    - postgresql_user: db=acme name=test priv=ALL/products:ALL state=absent fail_on_user=no
    
    # Remove test user from test database and the cluster
    - postgresql_user: db=test name=test priv=ALL state=absent
    
    # Example privileges string format
    INSERT,UPDATE/table:SELECT/anothertable:ALL
    
    # Remove an existing user's password
    - postgresql_user: db=test user=test password=NULL

> QUANTUM_FLOATING_IP
=======================

::

    Add or Remove a floating IP to an instance

    Options (= is mandatory):

    - auth_url
          The keystone url for authentication

    = instance_name
          The name of the instance to which the IP address should be
          assigned

    - internal_network_name
          The name of the network of the port to associate with the
          floating ip. Necessary when VM multiple networks.

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    = network_name
          Name of the network from which IP has to be assigned to VM.
          Please make sure the network is an external network

    - region_name
          Name of the region

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    Requirements:    novaclient, quantumclient, neutronclient, keystoneclient

    # Assign a floating ip to the instance from an external network
    - quantum_floating_ip: state=present login_username=admin login_password=admin
                           login_tenant_name=admin network_name=external_network
                           instance_name=vm1 internal_network_name=internal_network


> QUANTUM_NETWORK
=======================

::

    Add or Remove network from OpenStack.

    Options (= is mandatory):

    - admin_state_up
          Whether the state should be marked as up or down

    - auth_url
          The keystone url for authentication

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    = name
          Name to be assigned to the nework

    - provider_network_type
          The type of the network to be created, gre, vlan, local.
          Available types depend on the plugin. The Quantum service
          decides if not specified.

    - provider_physical_network
          The physical network which would realize the virtual network
          for flat and vlan networks.

    - provider_segmentation_id
          The id that has to be assigned to the network, in case of
          vlan networks that would be vlan id and for gre the tunnel
          id

    - region_name
          Name of the region

    - router_external
          If 'yes', specifies that the virtual network is a external
          network (public).

    - shared
          Whether this network is shared or not

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - tenant_name
          The name of the tenant for whom the network is created

    Requirements:    quantumclient, neutronclient, keystoneclient

    # Create a GRE backed Quantum network with tunnel id 1 for tenant1
    - quantum_network: name=t1network tenant_name=tenant1 state=present
                       provider_network_type=gre provider_segmentation_id=1
                       login_username=admin login_password=admin login_tenant_name=admin
    
    # Create an external network
    - quantum_network: name=external_network state=present
                       provider_network_type=local router_external=yes
                       login_username=admin login_password=admin login_tenant_name=admin

> QUANTUM_ROUTER
=======================

::

    Create or Delete routers from OpenStack

    Options (= is mandatory):

    - admin_state_up
          desired admin state of the created router .

    - auth_url
          The keystone url for authentication

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    = name
          Name to be give to the router

    - region_name
          Name of the region

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - tenant_name
          Name of the tenant for which the router has to be created,
          if none router would be created for the login tenant.

    Requirements:    quantumclient, neutronclient, keystoneclient

    # Creates a router for tenant admin
    - quantum_router: state=present
                    login_username=admin
                    login_password=admin
                    login_tenant_name=admin
                    name=router1"



> QUANTUM_SUBNET
=======================

::

    Add or Remove a floating IP to an instance

    Options (= is mandatory):

    - allocation_pool_end
          From the subnet pool the last IP that should be assigned to
          the virtual machines

    - allocation_pool_start
          From the subnet pool the starting address from which the IP
          should be allocated

    - auth_url
          The keystone URL for authentication

    = cidr
          The CIDR representation of the subnet that should be
          assigned to the subnet

    - dns_nameservers
          DNS nameservers for this subnet, comma-separated

    - enable_dhcp
          Whether DHCP should be enabled for this subnet.

    - gateway_ip
          The ip that would be assigned to the gateway for this subnet

    - ip_version
          The IP version of the subnet 4 or 6

    = login_password
          Password of login user

    = login_tenant_name
          The tenant name of the login user

    = login_username
          login username to authenticate to keystone

    = network_name
          Name of the network to which the subnet should be attached

    - region_name
          Name of the region

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - tenant_name
          The name of the tenant for whom the subnet should be created

    Requirements:    quantumclient, neutronclient, keystoneclient

    # Create a subnet for a tenant with the specified subnet
    - quantum_subnet: state=present login_username=admin login_password=admin
                      login_tenant_name=admin tenant_name=tenant1
                      network_name=network1 name=net1subnet cidr=192.168.0.0/24"

> RABBITMQ_PARAMETER
=======================

::

    Manage dynamic, cluster-wide parameters for RabbitMQ

    Options (= is mandatory):

    = component
          Name of the component of which the parameter is being set

    = name
          Name of the parameter being set

    - node
          erlang node name of the rabbit we wish to configure

    - state
          Specify if user is to be added or removed (Choices: present,
          absent)

    - value
          Value of the parameter, as a JSON term

    - vhost
          vhost to apply access privileges.

    # Set the federation parameter 'local_username' to a value of 'guest' (in quotes)
    - rabbitmq_parameter: component=federation
                          name=local-username
                          value='"guest"'
                          state=present

> RABBITMQ_PLUGIN
=======================

::

    Enables or disables RabbitMQ plugins

    Options (= is mandatory):

    = names
          Comma-separated list of plugin names

    - new_only
          Only enable missing pluginsDoes not disable plugins that are
          not in the names list (Choices: yes, no)

    - prefix
          Specify a custom install prefix to a Rabbit

    - state
          Specify if plugins are to be enabled or disabled (Choices:
          enabled, disabled)

    # Enables the rabbitmq_management plugin
    - rabbitmq_plugin: names=rabbitmq_management state=enabled

> RABBITMQ_POLICY
=======================

::

    Manage the state of a virtual host in RabbitMQ.

    Options (= is mandatory):

    = name
          The name of the policy to manage.

    - node
          Erlang node name of the rabbit we wish to configure.

    = pattern
          A regex of queues to apply the policy to.

    - priority
          The priority of the policy.

    - state
          The state of the policy. (Choices: present, absent)

    = tags
          A dict or string describing the policy.

    - vhost
          The name of the vhost to apply to.

    - name: ensure the default vhost contains the HA policy via a dict
      rabbitmq_policy: name=HA pattern='.*'
      args:
        tags:
          "ha-mode": all
    
    - name: ensure the default vhost contains the HA policy
      rabbitmq_policy: name=HA pattern='.*' tags="ha-mode=all"

> RABBITMQ_USER
=======================

::

    Add or remove users to RabbitMQ and assign permissions

    Options (= is mandatory):

    - configure_priv
          Regular expression to restrict configure actions on a
          resource for the specified vhost.By default all actions are
          restricted.

    - force
          Deletes and recreates the user. (Choices: yes, no)

    - node
          erlang node name of the rabbit we wish to configure

    - password
          Password of user to add.To change the password of an
          existing user, you must also specify `force=yes'.

    - read_priv
          Regular expression to restrict configure actions on a
          resource for the specified vhost.By default all actions are
          restricted.

    - state
          Specify if user is to be added or removed (Choices: present,
          absent)

    - tags
          User tags specified as comma delimited

    = user
          Name of user to add

    - vhost
          vhost to apply access privileges.

    - write_priv
          Regular expression to restrict configure actions on a
          resource for the specified vhost.By default all actions are
          restricted.

    # Add user to server and assign full access control
    - rabbitmq_user: user=joe
                     password=changeme
                     vhost=/
                     configure_priv=.*
                     read_priv=.*
                     write_priv=.*
                     state=present

> RABBITMQ_VHOST
=======================

::

    Manage the state of a virtual host in RabbitMQ

    Options (= is mandatory):

    = name
          The name of the vhost to manage

    - node
          erlang node name of the rabbit we wish to configure

    - state
          The state of vhost (Choices: present, absent)

    - tracing
          Enable/disable tracing for a vhost (Choices: yes, no)

    # Ensure that the vhost /test exists.
    - rabbitmq_vhost: name=/test state=present

> RAW
=======================

::

    Executes a low-down and dirty SSH command, not going through the
    module subsystem. This is useful and should only be done in two
    cases. The first case is installing `python-simplejson' on older
    (Python 2.4 and before) hosts that need it as a dependency to run
    modules, since nearly all core modules require it. Another is
    speaking to any devices such as routers that do not have any
    Python installed. In any other case, using the [shell] or
    [command] module is much more appropriate. Arguments given to
    [raw] are run directly through the configured remote shell.
    Standard output, error output and return code are returned when
    available. There is no change handler support for this module.This
    module does not require python on the remote system, much like the
    [script] module.

    Options (= is mandatory):

    - executable
          change the shell used to execute the command. Should be an
          absolute path to the executable.

    = free_form
          the raw module takes a free form command to run

    Notes:    If you want to execute a command securely and predictably, it may
          be better to use the [command] module instead. Best
          practices when writing playbooks will follow the trend of
          using [command] unless [shell] is explicitly required. When
          running ad-hoc commands, use your best judgement.

    # Bootstrap a legacy python 2.4 host
    - raw: yum -y install python-simplejson

> RAX
=======================

::

    creates / deletes a Rackspace Public Cloud instance and optionally
    waits for it to be 'running'.

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - auth_endpoint
          The URI of the authentication service

    - auto_increment
          Whether or not to increment a single number with the name of
          the created servers. Only applicable when used with the
          `group' attribute or meta key.

    - count
          number of instances to launch

    - count_offset
          number count to start at

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - disk_config
          Disk partitioning strategy (Choices: auto, manual)

    - env
          Environment as configured in ~/.pyrax.cfg, see https://githu
          b.com/rackspace/pyrax/blob/master/docs/getting_started.md
          #pyrax-configuration

    - exact_count
          Explicitly ensure an exact count of instances, used with
          state=active/present

    - files
          Files to insert into the instance.
          remotefilename:localcontent

    - flavor
          flavor to use for the instance

    - group
          host group to assign to server, is also used for idempotent
          operations to ensure a specific number of instances

    - identity_type
          Authentication machanism to use, such as rackspace or
          keystone

    - image
          image to use for the instance. Can be an `id', `human_id' or
          `name'

    - instance_ids
          list of instance ids, currently only used when
          state='absent' to remove instances

    - key_name
          key pair to use on the instance

    - meta
          A hash of metadata to associate with the instance

    - name
          Name to give the instance

    - networks
          The network to attach to the instances. If specified, you
          must include ALL networks including the public and private
          interfaces. Can be `id' or `label'.

    - region
          Region to create an instance in

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - tenant_id
          The tenant ID used for authentication

    - tenant_name
          The tenant name used for authentication

    - username
          Rackspace username (overrides `credentials')

    - verify_ssl
          Whether or not to require SSL validation of API endpoints

    - wait
          wait for the instance to be in state 'running' before
          returning (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: Build a Cloud Server
      gather_facts: False
      tasks:
        - name: Server build request
          local_action:
            module: rax
            credentials: ~/.raxpub
            name: rax-test1
            flavor: 5
            image: b11d9567-e412-4255-96b9-bd63ab23bcfe
            files:
              /root/.ssh/authorized_keys: /home/localuser/.ssh/id_rsa.pub
              /root/test.txt: /home/localuser/test.txt
            wait: yes
            state: present
            networks:
              - private
              - public
          register: rax
    
    - name: Build an exact count of cloud servers with incremented names
      hosts: local
      gather_facts: False
      tasks:
        - name: Server build requests
          local_action:
            module: rax
            credentials: ~/.raxpub
            name: test%03d.example.org
            flavor: performance1-1
            image: ubuntu-1204-lts-precise-pangolin
            state: present
            count: 10
            count_offset: 10
            exact_count: yes
            group: test
            wait: yes
          register: rax

> RAX_CLB
=======================

::

    creates / deletes a Rackspace Public Cloud load balancer.

    Options (= is mandatory):

    - algorithm
          algorithm for the balancer being created (Choices: RANDOM,
          LEAST_CONNECTIONS, ROUND_ROBIN, WEIGHTED_LEAST_CONNECTIONS,
          WEIGHTED_ROUND_ROBIN)

    - api_key
          Rackspace API key (overrides `credentials')

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - meta
          A hash of metadata to associate with the instance

    - name
          Name to give the load balancer

    - port
          Port for the balancer being created

    - protocol
          Protocol for the balancer being created (Choices: DNS_TCP,
          DNS_UDP, FTP, HTTP, HTTPS, IMAPS, IMAPv4, LDAP, LDAPS,
          MYSQL, POP3, POP3S, SMTP, TCP, TCP_CLIENT_FIRST, UDP,
          UDP_STREAM, SFTP)

    - region
          Region to create the load balancer in

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - timeout
          timeout for communication between the balancer and the node

    - type
          type of interface for the balancer being created (Choices:
          PUBLIC, SERVICENET)

    - username
          Rackspace username (overrides `credentials')

    - vip_id
          Virtual IP ID to use when creating the load balancer for
          purposes of sharing an IP with another load balancer of
          another protocol

    - wait
          wait for the balancer to be in state 'running' before
          returning (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: Build a Load Balancer
      gather_facts: False
      hosts: local
      connection: local
      tasks:
        - name: Load Balancer create request
          local_action:
            module: rax_clb
            credentials: ~/.raxpub
            name: my-lb
            port: 8080
            protocol: HTTP
            type: SERVICENET
            timeout: 30
            region: DFW
            wait: yes
            state: present
            meta:
              app: my-cool-app
          register: my_lb

> RAX_CLB_NODES
=======================

::

    Adds, modifies and removes nodes from a Rackspace Cloud Load
    Balancer

    Options (= is mandatory):

    - address
          IP address or domain name of the node

    - api_key
          Rackspace API key (overrides `credentials')

    - condition
          Condition for the node, which determines its role within the
          load balancer (Choices: enabled, disabled, draining)

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    = load_balancer_id
          Load balancer id

    - node_id
          Node id

    - port
          Port number of the load balanced service on the node

    - region
          Region to authenticate in

    - state
          Indicate desired state of the node (Choices: present,
          absent)

    - type
          Type of node (Choices: primary, secondary)

    - username
          Rackspace username (overrides `credentials')

    - virtualenv
          Path to a virtualenv that should be activated before doing
          anything. The virtualenv has to already exist. Useful if
          installing pyrax globally is not an option.

    - wait
          Wait for the load balancer to become active before returning
          (Choices: yes, no)

    - wait_timeout
          How long to wait before giving up and returning an error

    - weight
          Weight of node

    Notes:    The following environment variables can be used: `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDENTIALS' and `RAX_REGION'.

    Requirements:    pyrax

    # Add a new node to the load balancer
    - local_action:
        module: rax_clb_nodes
        load_balancer_id: 71
        address: 10.2.2.3
        port: 80
        condition: enabled
        type: primary
        wait: yes
        credentials: /path/to/credentials
    
    # Drain connections from a node
    - local_action:
        module: rax_clb_nodes
        load_balancer_id: 71
        node_id: 410
        condition: draining
        wait: yes
        credentials: /path/to/credentials
    
    # Remove a node from the load balancer
    - local_action:
        module: rax_clb_nodes
        load_balancer_id: 71
        node_id: 410
        state: absent
        wait: yes
        credentials: /path/to/credentials

> RAX_DNS_RECORD
=======================

::

    Manage DNS records on Rackspace Cloud DNS

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - comment
          Brief description of the domain. Maximum length of 160
          characters

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    = data
          IP address for A/AAAA record, FQDN for CNAME/MX/NS, or text
          data for SRV/TXT

    = domain
          Domain name to create the record in

    = name
          FQDN record name to create

    - priority
          Required for MX and SRV records, but forbidden for other
          record types. If specified, must be an integer from 0 to
          65535.

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - ttl
          Time to live of domain in seconds

    - type
          DNS record type (Choices: A, AAAA, CNAME, MX, NS, SRV, TXT)

    - username
          Rackspace username (overrides `credentials')

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: Create record
      hosts: all
      gather_facts: False
      tasks:
        - name: Record create request
          local_action:
            module: rax_dns_record
            credentials: ~/.raxpub
            domain: example.org
            name: www.example.org
            data: 127.0.0.1
            type: A
          register: rax_dns_record

> RAX_FACTS
=======================

::

    Gather facts for Rackspace Cloud Servers.

    Options (= is mandatory):

    - address
          Server IP address to retrieve facts for, will match any IP
          assigned to the server

    - api_key
          Rackspace API key (overrides `credentials')

    - auth_endpoint
          The URI of the authentication service

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - env
          Environment as configured in ~/.pyrax.cfg, see https://githu
          b.com/rackspace/pyrax/blob/master/docs/getting_started.md
          #pyrax-configuration

    - id
          Server ID to retrieve facts for

    - identity_type
          Authentication machanism to use, such as rackspace or
          keystone

    - name
          Server name to retrieve facts for

    - region
          Region to create an instance in

    - tenant_id
          The tenant ID used for authentication

    - tenant_name
          The tenant name used for authentication

    - username
          Rackspace username (overrides `credentials')

    - verify_ssl
          Whether or not to require SSL validation of API endpoints

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: Gather info about servers
      hosts: all
      gather_facts: False
      tasks:
        - name: Get facts about servers
          local_action:
            module: rax_facts
            credentials: ~/.raxpub
            name: "{{ inventory_hostname }}"
            region: DFW
        - name: Map some facts
          set_fact:
            ansible_ssh_host: "{{ rax_accessipv4 }}"

> RAX_FILES
=======================

::

    Manipulate Rackspace Cloud Files Containers

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - clear_meta
          Optionally clear existing metadata when applying metadata to
          existing containers. Selecting this option is only
          appropriate when setting type=meta (Choices: yes, no)

    = container
          The container to use for container or metadata operations.

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - meta
          A hash of items to set as metadata values on a container

    - private
          Used to set a container as private, removing it from the
          CDN.  *Warning!* Private containers, if previously made
          public, can have live objects available until the TTL on
          cached objects expires

    - public
          Used to set a container as public, available via the Cloud
          Files CDN

    - region
          Region to create an instance in

    - ttl
          In seconds, set a container-wide TTL for all objects cached
          on CDN edge nodes. Setting a TTL is only appropriate for
          containers that are public

    - type
          Type of object to do work on, i.e. metadata object or a
          container object (Choices: file, meta)

    - username
          Rackspace username (overrides `credentials')

    - web_error
          Sets an object to be presented as the HTTP error page when
          accessed by the CDN URL

    - web_index
          Sets an object to be presented as the HTTP index page when
          accessed by the CDN URL

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: "Test Cloud Files Containers"
      hosts: local
      gather_facts: no
      tasks:
        - name: "List all containers"
          rax_files: state=list
    
        - name: "Create container called 'mycontainer'"
          rax_files: container=mycontainer
    
        - name: "Create container 'mycontainer2' with metadata"
          rax_files:
            container: mycontainer2
            meta:
              key: value
              file_for: someuser@example.com
    
        - name: "Set a container's web index page"
          rax_files: container=mycontainer web_index=index.html
    
        - name: "Set a container's web error page"
          rax_files: container=mycontainer web_error=error.html
    
        - name: "Make container public"
          rax_files: container=mycontainer public=yes
    
        - name: "Make container public with a 24 hour TTL"
          rax_files: container=mycontainer public=yes ttl=86400
    
        - name: "Make container private"
          rax_files: container=mycontainer private=yes
    
    - name: "Test Cloud Files Containers Metadata Storage"
      hosts: local
      gather_facts: no
      tasks:
        - name: "Get mycontainer2 metadata"
          rax_files:
            container: mycontainer2
            type: meta
    
        - name: "Set mycontainer2 metadata"
          rax_files:
            container: mycontainer2
            type: meta
            meta:
              uploaded_by: someuser@example.com
    
        - name: "Remove mycontainer2 metadata"
          rax_files:
            container: "mycontainer2"
            type: meta
            state: absent
            meta:
              key: ""
              file_for: ""

> RAX_FILES_OBJECTS
=======================

::

    Upload, download, and delete objects in Rackspace Cloud Files

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - clear_meta
          Optionally clear existing metadata when applying metadata to
          existing objects. Selecting this option is only appropriate
          when setting type=meta (Choices: yes, no)

    = container
          The container to use for file object operations.

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - dest
          The destination of a "get" operation; i.e. a local
          directory, "/home/user/myfolder". Used to specify the
          destination of an operation on a remote object; i.e. a file
          name, "file1", or a comma-separated list of remote objects,
          "file1,file2,file17"

    - expires
          Used to set an expiration on a file or folder uploaded to
          Cloud Files. Requires an integer, specifying expiration in
          seconds

    - meta
          A hash of items to set as metadata values on an uploaded
          file or folder

    - method
          The method of operation to be performed.  For example, put
          to upload files to Cloud Files, get to download files from
          Cloud Files or delete to delete remote objects in Cloud
          Files (Choices: get, put, delete)

    - region
          Region in which to work.  Maps to a Rackspace Cloud region,
          i.e. DFW, ORD, IAD, SYD, LON

    - src
          Source from which to upload files.  Used to specify a remote
          object as a source for an operation, i.e. a file name,
          "file1", or a comma-separated list of remote objects,
          "file1,file2,file17".  src and dest are mutually exclusive
          on remote-only object operations

    - structure
          Used to specify whether to maintain nested directory
          structure when downloading objects from Cloud Files.
          Setting to false downloads the contents of a container to a
          single, flat directory (Choices: yes, no)

    - type
          Type of object to do work onMetadata object or a file object
          (Choices: file, meta)

    - username
          Rackspace username (overrides `credentials')

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: "Test Cloud Files Objects"
      hosts: local
      gather_facts: False
      tasks:
        - name: "Get objects from test container"
          rax_files_objects: container=testcont dest=~/Downloads/testcont
    
        - name: "Get single object from test container"
          rax_files_objects: container=testcont src=file1 dest=~/Downloads/testcont
    
        - name: "Get several objects from test container"
          rax_files_objects: container=testcont src=file1,file2,file3 dest=~/Downloads/testcont
    
        - name: "Delete one object in test container"
          rax_files_objects: container=testcont method=delete dest=file1
    
        - name: "Delete several objects in test container"
          rax_files_objects: container=testcont method=delete dest=file2,file3,file4
    
        - name: "Delete all objects in test container"
          rax_files_objects: container=testcont method=delete
    
        - name: "Upload all files to test container"
          rax_files_objects: container=testcont method=put src=~/Downloads/onehundred
    
        - name: "Upload one file to test container"
          rax_files_objects: container=testcont method=put src=~/Downloads/testcont/file1
    
        - name: "Upload one file to test container with metadata"
          rax_files_objects:
            container: testcont
            src: ~/Downloads/testcont/file2
            method: put
            meta:
              testkey: testdata
              who_uploaded_this: someuser@example.com
    
        - name: "Upload one file to test container with TTL of 60 seconds"
          rax_files_objects: container=testcont method=put src=~/Downloads/testcont/file3 expires=60
    
        - name: "Attempt to get remote object that does not exist"
          rax_files_objects: container=testcont method=get src=FileThatDoesNotExist.jpg dest=~/Downloads/testcont
          ignore_errors: yes
    
        - name: "Attempt to delete remote object that does not exist"
          rax_files_objects: container=testcont method=delete dest=FileThatDoesNotExist.jpg
          ignore_errors: yes
    
    - name: "Test Cloud Files Objects Metadata"
      hosts: local
      gather_facts: false
      tasks:
        - name: "Get metadata on one object"
          rax_files_objects:  container=testcont type=meta dest=file2
    
        - name: "Get metadata on several objects"
          rax_files_objects:  container=testcont type=meta src=file2,file1
    
        - name: "Set metadata on an object"
          rax_files_objects:
            container: testcont
            type: meta
            dest: file17
            method: put
            meta:
              key1: value1
              key2: value2
            clear_meta: true
    
        - name: "Verify metadata is set"
          rax_files_objects:  container=testcont type=meta src=file17
    
        - name: "Delete metadata"
          rax_files_objects:
            container: testcont
            type: meta
            dest: file17
            method: delete
            meta:
              key1: ''
              key2: ''
    
        - name: "Get metadata on all objects"
          rax_files_objects:  container=testcont type=meta

> RAX_KEYPAIR
=======================

::

    Create a keypair for use with Rackspace Cloud Servers

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - auth_endpoint
          The URI of the authentication service

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - env
          Environment as configured in ~/.pyrax.cfg, see https://githu
          b.com/rackspace/pyrax/blob/master/docs/getting_started.md
          #pyrax-configuration

    - identity_type
          Authentication machanism to use, such as rackspace or
          keystone

    = name
          Name of keypair

    - public_key
          Public Key string to upload

    - region
          Region to create an instance in

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - tenant_id
          The tenant ID used for authentication

    - tenant_name
          The tenant name used for authentication

    - username
          Rackspace username (overrides `credentials')

    - verify_ssl
          Whether or not to require SSL validation of API endpoints

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)Keypairs cannot be
          manipulated, only created and deleted. To "update" a keypair
          you must first delete and then recreate.

    Requirements:    pyrax

    - name: Create a keypair
      hosts: local
      gather_facts: False
      tasks:
        - name: keypair request
          local_action:
            module: rax_keypair
            credentials: ~/.raxpub
            name: my_keypair
            region: DFW
          register: keypair
        - name: Create local public key
          local_action:
            module: copy
            content: "{{ keypair.keypair.public_key }}"
            dest: "{{ inventory_dir }}/{{ keypair.keypair.name }}.pub"
        - name: Create local private key
          local_action:
            module: copy
            content: "{{ keypair.keypair.private_key }}"
            dest: "{{ inventory_dir }}/{{ keypair.keypair.name }}"

> RAX_NETWORK
=======================

::

    creates / deletes a Rackspace Public Cloud isolated network.

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - cidr
          cidr of the network being created

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - label
          Label (name) to give the network

    - region
          Region to create the network in

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - username
          Rackspace username (overrides `credentials')

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS' points to a
          credentials file appropriate for pyrax`RAX_USERNAME' and
          `RAX_API_KEY' obviate the use of a credentials
          file`RAX_REGION' defines a Rackspace Public Cloud region
          (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: Build an Isolated Network
      gather_facts: False
    
      tasks:
        - name: Network create request
          local_action:
            module: rax_network
            credentials: ~/.raxpub
            label: my-net
            cidr: 192.168.3.0/24
            state: present

> RAX_QUEUE
=======================

::

    creates / deletes a Rackspace Public Cloud queue.

    Options (= is mandatory):

    - api_key
          Rackspace API key (overrides `credentials')

    - credentials
          File to find the Rackspace credentials in (ignored if
          `api_key' and `username' are provided)

    - name
          Name to give the queue

    - region
          Region to create the load balancer in

    - state
          Indicate desired state of the resource (Choices: present,
          absent)

    - username
          Rackspace username (overrides `credentials')

    Notes:    The following environment variables can be used, `RAX_USERNAME',
          `RAX_API_KEY', `RAX_CREDS_FILE', `RAX_CREDENTIALS',
          `RAX_REGION'.`RAX_CREDENTIALS' and `RAX_CREDS_FILE' points
          to a credentials file appropriate for pyrax. See https://git
          hub.com/rackspace/pyrax/blob/master/docs/getting_started.md#
          authenticating`RAX_USERNAME' and `RAX_API_KEY' obviate the
          use of a credentials file`RAX_REGION' defines a Rackspace
          Public Cloud region (DFW, ORD, LON, ...)

    Requirements:    pyrax

    - name: Build a Queue
      gather_facts: False
      hosts: local
      connection: local
      tasks:
        - name: Queue create request
          local_action:
            module: rax_queue
            credentials: ~/.raxpub
            client_id: unique-client-name
            name: my-queue
            region: DFW
            state: present
          register: my_queue

> RDS
=======================

::

    Creates, deletes, or modifies rds instances.  When creating an
    instance it can be either a new instance or a read-only replica of
    an existing instance. This module has a dependency on python-boto
    >= 2.5. The 'promote' command requires boto >= 2.18.0.

    Options (= is mandatory):

    - apply_immediately
          Used only when command=modify.  If enabled, the
          modifications will be applied as soon as possible rather
          than waiting for the next preferred maintenance window.
          (Choices: yes, no)

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    - backup_retention
          Number of days backups are retained.  Set to 0 to disable
          backups.  Default is 1 day.  Valid range: 0-35. Used only
          when command=create or command=modify.

    - backup_window
          Backup window in format of hh24:mi-hh24:mi.  If not
          specified then a random backup window is assigned. Used only
          when command=create or command=modify.

    = command
          Specifies the action to take. (Choices: create, replicate,
          delete, facts, modify, promote, snapshot, restore)

    - db_engine
          The type of database.  Used only when command=create.
          (Choices: MySQL, oracle-se1, oracle-se, oracle-ee,
          sqlserver-ee, sqlserver-se, sqlserver-ex, sqlserver-web,
          postgres)

    - db_name
          Name of a database to create within the instance.  If not
          specified then no database is created. Used only when
          command=create.

    - engine_version
          Version number of the database engine to use. Used only when
          command=create. If not specified then the current Amazon RDS
          default engine version is used.

    = instance_name
          Database instance identifier.

    - instance_type
          The instance type of the database.  Must be specified when
          command=create. Optional when command=replicate,
          command=modify or command=restore. If not specified then the
          replica inherits the same instance type as the source
          instance. (Choices: db.t1.micro, db.m1.small, db.m1.medium,
          db.m1.large, db.m1.xlarge, db.m2.xlarge, db.m2.2xlarge,
          db.m2.4xlarge)

    - iops
          Specifies the number of IOPS for the instance.  Used only
          when command=create or command=modify. Must be an integer
          greater than 1000.

    - license_model
          The license model for this DB instance. Used only when
          command=create or command=restore. (Choices: license-
          included, bring-your-own-license, general-public-license)

    - maint_window
          Maintenance window in format of ddd:hh24:mi-ddd:hh24:mi.
          (Example: Mon:22:00-Mon:23:15) If not specified then a
          random maintenance window is assigned. Used only when
          command=create or command=modify.

    - multi_zone
          Specifies if this is a Multi-availability-zone deployment.
          Can not be used in conjunction with zone parameter. Used
          only when command=create or command=modify. (Choices: yes,
          no)

    - new_instance_name
          Name to rename an instance to. Used only when
          command=modify.

    - option_group
          The name of the option group to use.  If not specified then
          the default option group is used. Used only when
          command=create.

    - parameter_group
          Name of the DB parameter group to associate with this
          instance.  If omitted then the RDS default DBParameterGroup
          will be used. Used only when command=create or
          command=modify.

    - password
          Password for the master database username. Used only when
          command=create or command=modify.

    - port
          Port number that the DB instance uses for connections.
          Defaults to 3306 for mysql, 1521 for Oracle, 1443 for SQL
          Server. Used only when command=create or command=replicate.

    = region
          The AWS region to use. If not specified then the value of
          the EC2_REGION environment variable, if any, is used.

    - security_groups
          Comma separated list of one or more security groups.  Used
          only when command=create or command=modify.

    - size
          Size in gigabytes of the initial storage for the DB
          instance. Used only when command=create or command=modify.

    - snapshot
          Name of snapshot to take. When command=delete, if no
          snapshot name is provided then no snapshot is taken. Used
          only when command=delete or command=snapshot.

    - source_instance
          Name of the database to replicate. Used only when
          command=replicate.

    - subnet
          VPC subnet group.  If specified then a VPC instance is
          created. Used only when command=create.

    - upgrade
          Indicates that minor version upgrades should be applied
          automatically. Used only when command=create or
          command=replicate. (Choices: yes, no)

    - username
          Master database username. Used only when command=create.

    - vpc_security_groups
          Comma separated list of one or more vpc security groups.
          Used only when command=create or command=modify.

    - wait
          When command=create, replicate, modify or restore then wait
          for the database to enter the 'available' state.  When
          command=delete wait for the database to be terminated.
          (Choices: yes, no)

    - wait_timeout
          how long before wait gives up, in seconds

    - zone
          availability zone in which to launch the instance. Used only
          when command=create, command=replicate or command=restore.

    Requirements:    boto

    # Basic mysql provisioning example
    - rds: >
          command=create
          instance_name=new_database
          db_engine=MySQL
          size=10
          instance_type=db.m1.small
          username=mysql_admin
          password=1nsecure
    
    # Create a read-only replica and wait for it to become available
    - rds: >
          command=replicate
          instance_name=new_database_replica
          source_instance=new_database
          wait=yes
          wait_timeout=600
    
    # Delete an instance, but create a snapshot before doing so
    - rds: >
          command=delete
          instance_name=new_database
          snapshot=new_database_snapshot
    
    # Get facts about an instance
    - rds: >
          command=facts
          instance_name=new_database
          register: new_database_facts
    
    # Rename an instance and wait for the change to take effect
    - rds: >
          command=modify
          instance_name=new_database
          new_instance_name=renamed_database
          wait=yes
        

> REDHAT_SUBSCRIPTION
=======================

::

    Manage registration and subscription to the Red Hat Network
    entitlement platform.

    Options (= is mandatory):

    - activationkey
          supply an activation key for use with registration

    - autosubscribe
          Upon successful registration, auto-consume available
          subscriptions

    - password
          Red Hat Network password

    - pool
          Specify a subscription pool name to consume.  Regular
          expressions accepted.

    - rhsm_baseurl
          Specify CDN baseurl

    - server_hostname
          Specify an alternative Red Hat Network server

    - server_insecure
          Allow traffic over insecure http

    - state
          whether to register and subscribe (`present'), or unregister
          (`absent') a system (Choices: present, absent)

    - username
          Red Hat Network username

    Notes:    In order to register a system, subscription-manager requires
          either a username and password, or an activationkey.

    Requirements:    subscription-manager

    # Register as user (joe_user) with password (somepass) and auto-subscribe to available content.
    - redhat_subscription: action=register username=joe_user password=somepass autosubscribe=true
    
    # Register with activationkey (1-222333444) and consume subscriptions matching
    # the names (Red hat Enterprise Server) and (Red Hat Virtualization)
    - redhat_subscription: action=register
                           activationkey=1-222333444
                           pool='^(Red Hat Enterprise Server|Red Hat Virtualization)$'

> REDIS
=======================

::

    Unified utility to interact with redis instances. 'slave' Sets a
    redis instance in slave or master mode. 'flush' Flushes all the
    instance or a specified db.

    Options (= is mandatory):

    = command
          The selected redis command (Choices: slave, flush)

    - db
          The database to flush (used in db mode) [flush command]

    - flush_mode
          Type of flush (all the dbs in a redis instance or a specific
          one) [flush command] (Choices: all, db)

    - login_host
          The host running the database

    - login_password
          The password used to authenticate with (usually not used)

    - login_port
          The port to connect to

    - master_host
          The host of the master instance [slave command]

    - master_port
          The port of the master instance [slave command]

    - slave_mode
          the mode of the redis instance [slave command] (Choices:
          master, slave)

    Notes:    Requires the redis-py Python package on the remote host. You can
          install it with pip (pip install redis) or with a package
          manager. https://github.com/andymccurdy/redis-pyIf the redis
          master instance we are making slave of is password protected
          this needs to be in the redis.conf in the masterauth
          variable

    Requirements:    redis

    # Set local redis instance to be slave of melee.island on port 6377
    - redis: command=slave master_host=melee.island master_port=6377
    
    # Deactivate slave mode
    - redis: command=slave slave_mode=master
    
    # Flush all the redis db
    - redis: command=flush flush_mode=all
    
    # Flush only one db in a redis instance
    - redis: command=flush db=1 flush_mode=db

> RHN_CHANNEL
=======================

::

    Adds or removes Red Hat software channels

    Options (= is mandatory):

    = name
          name of the software channel

    = password
          the user's password

    - state
          whether the channel should be present or not

    = sysname
          name of the system as it is known in RHN/Satellite

    = url
          The full url to the RHN/Satellite api

    = user
          RHN/Satellite user

    Notes:    this module fetches the system id from RHN.

    Requirements:    none

    - rhn_channel: name=rhel-x86_64-server-v2vwin-6 sysname=server01 url=https://rhn.redhat.com/rpc/api user=rhnuser password=guessme

> RHN_REGISTER
=======================

::

    Manage registration to the Red Hat Network.

    Options (= is mandatory):

    - activationkey
          supply an activation key for use with registration

    - channels
          Optionally specify a list of comma-separated channels to
          subscribe to upon successful registration.

    - password
          Red Hat Network password

    - server_url
          Specify an alternative Red Hat Network server URL

    - state
          whether to register (`present'), or unregister (`absent') a
          system (Choices: present, absent)

    - username
          Red Hat Network username

    Notes:    In order to register a system, rhnreg_ks requires either a
          username and password, or an activationkey.

    Requirements:    rhnreg_ks

    # Unregister system from RHN.
    - rhn_register: state=absent username=joe_user password=somepass
    
    # Register as user (joe_user) with password (somepass) and auto-subscribe to available content.
    - rhn_register: state=present username=joe_user password=somepass
    
    # Register with activationkey (1-222333444) and enable extended update support.
    - rhn_register: state=present activationkey=1-222333444 enable_eus=true
    
    # Register as user (joe_user) with password (somepass) against a satellite
    # server specified by (server_url).
    - rhn_register:
        state=present
        username=joe_user
        password=somepass
        server_url=https://xmlrpc.my.satellite/XMLRPC
    
    # Register as user (joe_user) with password (somepass) and enable
    # channels (rhel-x86_64-server-6-foo-1) and (rhel-x86_64-server-6-bar-1).
    - rhn_register: state=present username=joe_user
                    password=somepass
                    channels=rhel-x86_64-server-6-foo-1,rhel-x86_64-server-6-bar-1

> RIAK
=======================

::

    This module can be used to join nodes to a cluster, check the
    status of the cluster.

    Options (= is mandatory):

    - command
          The command you would like to perform against the cluster.
          (Choices: ping, kv_test, join, plan, commit)

    - config_dir
          The path to the riak configuration directory

    - http_conn
          The ip address and port that is listening for Riak HTTP
          queries

    - target_node
          The target node for certain operations (join, ping)

    - validate_certs
          If `no', SSL certificates will not be validated. This should
          only be used on personally controlled sites using self-
          signed certificates. (Choices: yes, no)

    - wait_for_handoffs
          Number of seconds to wait for handoffs to complete.

    - wait_for_ring
          Number of seconds to wait for all nodes to agree on the
          ring.

    - wait_for_service
          Waits for a riak service to come online before continuing.
          (Choices: kv)

    # Join's a Riak node to another node
    - riak: command=join target_node=riak@10.1.1.1
    
    # Wait for handoffs to finish.  Use with async and poll.
    - riak: wait_for_handoffs=yes
    
    # Wait for riak_kv service to startup
    - riak: wait_for_service=kv

> ROUTE53
=======================

::

    Creates and deletes DNS records in Amazons Route53 service

    Options (= is mandatory):

    - aws_access_key
          AWS access key.

    - aws_secret_key
          AWS secret key.

    = command
          Specifies the action to take. (Choices: get, create, delete)

    - overwrite
          Whether an existing record should be overwritten on create
          if values do not match

    = record
          The full DNS record to create or delete

    - ttl
          The TTL to give the new record

    = type
          The type of DNS record to create (Choices: A, CNAME, MX,
          AAAA, TXT, PTR, SRV, SPF, NS)

    - value
          The new value when creating a DNS record.  Multiple comma-
          spaced values are allowed.  When deleting a record all
          values for the record must be specified or Route53 will not
          delete it.

    = zone
          The DNS zone to modify

    Requirements:    boto

    # Add new.foo.com as an A record with 3 IPs
    - route53: >
          command=create
          zone=foo.com
          record=new.foo.com
          type=A
          ttl=7200
          value=1.1.1.1,2.2.2.2,3.3.3.3
    
    # Retrieve the details for new.foo.com
    - route53: >
          command=get
          zone=foo.com
          record=new.foo.com
          type=A
      register: rec
    
    # Delete new.foo.com A record using the results from the get command
    - route53: >
          command=delete
          zone=foo.com
          record={{ rec.set.record }}
          type={{ rec.set.type }}
          value={{ rec.set.value }}
    
    # Add an AAAA record.  Note that because there are colons in the value
    # that the entire parameter list must be quoted:
    - route53: >
          command=create
          zone=foo.com
          record=localhost.foo.com
          type=AAAA
          ttl=7200
          value="::1"
    
    # Add a TXT record. Note that TXT and SPF records must be surrounded
    # by quotes when sent to Route 53:
    - route53: >
          command=create
          zone=foo.com
          record=localhost.foo.com
          type=TXT
          ttl=7200
          value=""bar""
    
    

> RPM_KEY
=======================

::

    Adds or removes (rpm --import) a gpg key to your rpm database.

    Options (= is mandatory):

    = key
          Key that will be modified. Can be a url, a file, or a keyid
          if the key already exists in the database.

    - state
          Wheather the key will be imported or removed from the rpm
          db. (Choices: present, absent)

    - validate_certs
          If `no' and the `key' is a url starting with https, SSL
          certificates will not be validated. This should only be used
          on personally controlled sites using self-signed
          certificates. (Choices: yes, no)

    # Example action to import a key from a url
    - rpm_key: state=present key=http://apt.sw.be/RPM-GPG-KEY.dag.txt
    
    # Example action to import a key from a file
    - rpm_key: state=present key=/path/to/key.gpg
    
    # Example action to ensure a key is not present in the db
    - rpm_key: state=absent key=DEADB33F

> S3
=======================

::

    This module allows the user to dictate the presence of a given
    file in an S3 bucket. If or once the key (file) exists in the
    bucket, it returns a time-expired download URL. This module has a
    dependency on python-boto.

    Options (= is mandatory):

    - aws_access_key
          AWS access key. If not set then the value of the
          AWS_ACCESS_KEY environment variable is used.

    - aws_secret_key
          AWS secret key. If not set then the value of the
          AWS_SECRET_KEY environment variable is used.

    = bucket
          Bucket name.

    - dest
          The destination file path when downloading an object/key
          with a GET operation.

    - expiration
          Time limit (in seconds) for the URL generated and returned
          by S3/Walrus when performing a mode=put or mode=geturl
          operation.

    = mode
          Switches the module behaviour between put (upload), get
          (download), geturl (return download url (Ansible 1.3+),
          getstr (download object as string (1.3+)), create (bucket)
          and delete (bucket).

    - object
          Keyname of the object inside the bucket. Can be used to
          create "virtual directories", see examples.

    - overwrite
          Force overwrite either locally on the filesystem or remotely
          with the object/key. Used with PUT and GET operations.

    - s3_url
          S3 URL endpoint. If not specified then the S3_URL
          environment variable is used, if that variable is defined.

    - src
          The source file path when performing a PUT operation.

    Requirements:    boto

    # Simple PUT operation
    - s3: bucket=mybucket object=/my/desired/key.txt src=/usr/local/myfile.txt mode=put
    # Simple GET operation
    - s3: bucket=mybucket object=/my/desired/key.txt dest=/usr/local/myfile.txt mode=get
    # GET/download and overwrite local file (trust remote)
    - s3: bucket=mybucket object=/my/desired/key.txt dest=/usr/local/myfile.txt mode=get 
    # GET/download and do not overwrite local file (trust remote)
    - s3: bucket=mybucket object=/my/desired/key.txt dest=/usr/local/myfile.txt mode=get force=false
    # PUT/upload and overwrite remote file (trust local)
    - s3: bucket=mybucket object=/my/desired/key.txt src=/usr/local/myfile.txt mode=put 
    # PUT/upload and do not overwrite remote file (trust local)
    - s3: bucket=mybucket object=/my/desired/key.txt src=/usr/local/myfile.txt mode=put force=false
    # Download an object as a string to use else where in your playbook
    - s3: bucket=mybucket object=/my/desired/key.txt src=/usr/local/myfile.txt mode=getstr
    # Create an empty bucket
    - s3: bucket=mybucket mode=create
    # Create a bucket with key as directory
    - s3: bucket=mybucket object=/my/directory/path mode=create
    # Delete a bucket and all contents
    - s3: bucket=mybucket mode=delete

> SCRIPT
=======================

::

    The [script] module takes the script name followed by a list of
    space-delimited arguments. The local script at path will be
    transfered to the remote node and then executed. The given script
    will be processed through the shell environment on the remote
    node. This module does not require python on the remote system,
    much like the [raw] module.

    Options (= is mandatory):

    - creates
          a filename, when it already exists, this step will *not* be
          run.

    = free_form
          path to the local script file followed by optional
          arguments.

    - removes
          a filename, when it does not exist, this step will *not* be
          run.

    Notes:    It is usually preferable to write Ansible modules than pushing
          scripts. Convert your script to an Ansible module for bonus
          points!

    # Example from Ansible Playbooks
    - script: /some/local/script.sh --some-arguments 1234
    
    # Run a script that creates a file, but only if the file is not yet created
    - script: /some/local/create_file.sh --some-arguments 1234 creates=/the/created/file.txt
    
    # Run a script that removes a file, but only if the file is not yet removed
    - script: /some/local/remove_file.sh --some-arguments 1234 removes=/the/removed/file.txt

> SEBOOLEAN
=======================

::

    Toggles SELinux booleans.

    Options (= is mandatory):

    = name
          Name of the boolean to configure

    - persistent
          Set to `yes' if the boolean setting should survive a reboot
          (Choices: yes, no)

    = state
          Desired boolean value (Choices: yes, no)

    Notes:    Not tested on any debian based system

    # Set (httpd_can_network_connect) flag on and keep it persistent across reboots
    - seboolean: name=httpd_can_network_connect state=yes persistent=yes

> SELINUX
=======================

::

    Configures the SELinux mode and policy. A reboot may be required
    after usage. Ansible will not issue this reboot but will let you
    know when it is required.

    Options (= is mandatory):

    - conf
          path to the SELinux configuration file, if non-standard

    - policy
          name of the SELinux policy to use (example: `targeted') will
          be required if state is not `disabled'

    = state
          The SELinux mode (Choices: enforcing, permissive, disabled)

    Notes:    Not tested on any debian based system

    Requirements:    libselinux-python

    - selinux: policy=targeted state=enforcing
    - selinux: policy=targeted state=permissive
    - selinux: state=disabled

> SERVICE
=======================

::

    Controls services on remote hosts.

    Options (= is mandatory):

    - arguments
          Additional arguments provided on the command line

    - enabled
          Whether the service should start on boot. At least one of
          state and enabled are required. (Choices: yes, no)

    = name
          Name of the service.

    - pattern
          If the service does not respond to the status command, name
          a substring to look for as would be found in the output of
          the `ps' command as a stand-in for a status result.  If the
          string is found, the service will be assumed to be running.

    - runlevel
          For OpenRC init scripts (ex: Gentoo) only.  The runlevel
          that this service belongs to.

    - sleep
          If the service is being `restarted' then sleep this many
          seconds between the stop and start command. This helps to
          workaround badly behaving init scripts that exit immediately
          after signaling a process to stop.

    - state
          `started'/`stopped' are idempotent actions that will not run
          commands unless necessary.  `restarted' will always bounce
          the service.  `reloaded' will always reload. At least one of
          state and enabled are required. (Choices: started, stopped,
          restarted, reloaded)

    # Example action to start service httpd, if not running
    - service: name=httpd state=started
    
    # Example action to stop service httpd, if running
    - service: name=httpd state=stopped
    
    # Example action to restart service httpd, in all cases
    - service: name=httpd state=restarted
    
    # Example action to reload service httpd, in all cases
    - service: name=httpd state=reloaded
    
    # Example action to enable service httpd, and not touch the running state
    - service: name=httpd enabled=yes
    
    # Example action to start service foo, based on running process /usr/bin/foo
    - service: name=foo pattern=/usr/bin/foo state=started
    
    # Example action to restart network service for interface eth0
    - service: name=network state=restarted args=eth0

> SET_FACT
=======================

::

    This module allows setting new variables.  Variables are set on a
    host-by-host basis just like facts discovered by the setup
    module.These variables will survive between plays.

    Options (= is mandatory):

    = key_value
          The `set_fact' module takes key=value pairs as variables to
          set in the playbook scope. Or alternatively, accepts complex
          arguments using the `args:' statement.

    # Example setting host facts using key=value pairs
    - set_fact: one_fact="something" other_fact="{{ local_var * 2 }}"
    
    # Example setting host facts using complex arguments
    - set_fact:
         one_fact: something
         other_fact: "{{ local_var * 2 }}"

> SETUP
=======================

::

    This module is automatically called by playbooks to gather useful
    variables about remote hosts that can be used in playbooks. It can
    also be executed directly by `/usr/bin/ansible' to check what
    variables are available to a host. Ansible provides many `facts'
    about the system, automatically.

    Options (= is mandatory):

    - fact_path
          path used for local ansible facts (*.fact) - files in this
          dir will be run (if executable) and their results be added
          to ansible_local facts if a file is not executable it is
          read. File/results format can be json or ini-format

    - filter
          if supplied, only return facts that match this shell-style
          (fnmatch) wildcard.

    Notes:    More ansible facts will be added with successive releases. If
          `facter' or `ohai' are installed, variables from these
          programs will also be snapshotted into the JSON file for
          usage in templating. These variables are prefixed with
          `facter_' and `ohai_' so it's easy to tell their source. All
          variables are bubbled up to the caller. Using the ansible
          facts and choosing to not install `facter' and `ohai' means
          you can avoid Ruby-dependencies on your remote systems. (See
          also [facter] and [ohai].)The filter option filters only the
          first level subkey below ansible_facts.

    # Display facts from all hosts and store them indexed by I(hostname) at C(/tmp/facts).
    ansible all -m setup --tree /tmp/facts
    
    # Display only facts regarding memory found by ansible on all hosts and output them.
    ansible all -m setup -a 'filter=ansible_*_mb'
    
    # Display only facts returned by facter.
    ansible all -m setup -a 'filter=facter_*'
    
    # Display only facts about certain interfaces.
    ansible all -m setup -a 'filter=ansible_eth[0-2]'

> SHELL
=======================

::

    The [shell] module takes the command name followed by a list of
    space-delimited arguments. It is almost exactly like the [command]
    module but runs the command through a shell (`/bin/sh') on the
    remote node.

    Options (= is mandatory):

    - chdir
          cd into this directory before running the command

    - creates
          a filename, when it already exists, this step will *not* be
          run.

    - executable
          change the shell used to execute the command. Should be an
          absolute path to the executable.

    = free_form
          The shell module takes a free form command to run

    - removes
          a filename, when it does not exist, this step will *not* be
          run.

    Notes:    If you want to execute a command securely and predictably, it may
          be better to use the [command] module instead. Best
          practices when writing playbooks will follow the trend of
          using [command] unless [shell] is explicitly required. When
          running ad-hoc commands, use your best judgement.

    # Execute the command in remote shell; stdout goes to the specified
    # file on the remote
    - shell: somescript.sh >> somelog.txt

> SLURP
=======================

::

    This module works like [fetch]. It is used for fetching a base64-
    encoded blob containing the data in a remote file.

    Options (= is mandatory):

    = src
          The file on the remote system to fetch. This `must' be a
          file, not a directory.

    Notes:    See also: [fetch]

    ansible host -m slurp -a 'src=/tmp/xx'
       host | success >> {
          "content": "aGVsbG8gQW5zaWJsZSB3b3JsZAo=", 
          "encoding": "base64"
       }

> STAT
=======================

::

    Retrieves facts for a file similar to the linux/unix 'stat'
    command.

    Options (= is mandatory):

    - follow
          Whether to follow symlinks

    - get_md5
          Whether to return the md5 sum of the file

    = path
          The full path of the file/object to get the facts of

    # Obtain the stats of /etc/foo.conf, and check that the file still belongs
    # to 'root'. Fail otherwise.
    - stat: path=/etc/foo.conf
      register: st
    - fail: msg="Whoops! file ownership has changed"
      when: st.stat.pw_name != 'root'
    
    # Determine if a path exists and is a directory.  Note we need to test
    # both that p.stat.isdir actually exists, and also that it's set to true.
    - stat: path=/path/to/something
      register: p
    - debug: msg="Path exists and is a directory"
      when: p.stat.isdir is defined and p.stat.isdir == true
    
    # Don't do md5 checksum
    - stat: path=/path/to/myhugefile get_md5=no

> SUBVERSION
=======================

::

    Deploy given repository URL / revision to dest. If dest exists,
    update to the specified revision, otherwise perform a checkout.

    Options (= is mandatory):

    = dest
          Absolute path where the repository should be deployed.

    - executable
          Path to svn executable to use. If not supplied, the normal
          mechanism for resolving binary paths will be used.

    - force
          If `yes', modified files will be discarded. If `no', module
          will fail if it encounters modified files. (Choices: yes,
          no)

    - password
          --password parameter passed to svn.

    = repo
          The subversion URL to the repository.

    - revision
          Specific revision to checkout.

    - username
          --username parameter passed to svn.

    Notes:    Requres `svn' to be installed on the client.

    # Checkout subversion repository to specified folder.
    - subversion: repo=svn+ssh://an.example.org/path/to/repo dest=/src/checkout

> SUPERVISORCTL
=======================

::

    Manage the state of a program or group of programs running via
    `Supervisord'

    Options (= is mandatory):

    - config
          configuration file path, passed as -c to supervisorctl

    = name
          The name of the `supervisord' program/process to manage

    - password
          password to use for authentication with server, passed as -p
          to supervisorctl

    - server_url
          URL on which supervisord server is listening, passed as -s
          to supervisorctl

    = state
          The state of service (Choices: present, started, stopped,
          restarted)

    - supervisorctl_path
          Path to supervisorctl executable to use

    - username
          username to use for authentication with server, passed as -u
          to supervisorctl

    # Manage the state of program to be in 'started' state.
    - supervisorctl: name=my_app state=started
    
    # Restart my_app, reading supervisorctl configuration from a specified file.
    - supervisorctl: name=my_app state=restarted config=/var/opt/my_project/supervisord.conf
    
    # Restart my_app, connecting to supervisord with credentials and server URL.
    - supervisorctl: name=my_app state=restarted username=test password=testpass server_url=http://localhost:9001
    

> SVR4PKG
=======================

::

    Manages SVR4 packages on Solaris 10 and 11.These were the native
    packages on Solaris <= 10 and are available as a legacy feature in
    Solaris 11.Note that this is a very basic packaging system. It
    will not enforce dependencies on install or remove.

    Options (= is mandatory):

    = name
          Package name, e.g. `SUNWcsr'

    - proxy
          HTTP[s] proxy to be used if `src' is a URL.

    - response_file
          Specifies the location of a response file to be used if
          package expects input on install. (added in Ansible 1.4)

    - src
          Specifies the location to install the package from. Required
          when `state=present'.Can be any path acceptable to the
          `pkgadd' command's `-d' option. e.g.: `somefile.pkg',
          `/dir/with/pkgs', `http:/server/mypkgs.pkg'.If using a file
          or directory, they must already be accessible by the host.
          See the [copy] module for a way to get them there.

    = state
          Whether to install (`present'), or remove (`absent') a
          package.If the package is to be installed, then `src' is
          required.The SVR4 package system doesn't provide an upgrade
          operation. You need to uninstall the old, then install the
          new package. (Choices: present, absent)

    # Install a package from an already copied file
    - svr4pkg: name=CSWcommon src=/tmp/cswpkgs.pkg state=present
    
    # Install a package directly from an http site
    - svr4pkg: name=CSWpkgutil src=http://get.opencsw.org/now state=present
    
    # Install a package with a response file
    - svr4pkg: name=CSWggrep src=/tmp/third-party.pkg response_file=/tmp/ggrep.response state=present
    
    # Ensure that a package is not installed.
    - svr4pkg: name=SUNWgnome-sound-recorder state=absent

> SWDEPOT
=======================

::

    Will install, upgrade and remove packages with swdepot package
    manager (HP-UX)

    Options (= is mandatory):

    - depot
          The source repository from which install or upgrade a
          package. (Choices: )

    = name
          package name. (Choices: )

    = state
          whether to install (`present', `latest'), or remove
          (`absent') a package. (Choices: present, latest, absent)

    - swdepot: name=unzip-6.0 state=installed depot=repository:/path
    - swdepot: name=unzip state=latest depot=repository:/path
    - swdepot: name=unzip state=absent

> SYNCHRONIZE
=======================

::

    This is a wrapper around rsync. Of course you could just use the
    command action to call rsync yourself, but you also have to add a
    fair number of boilerplate options and host facts. You still may
    need to call rsync directly via `command' or `shell' depending on
    your use case. The synchronize action is meant to do common things
    with `rsync' easily. It does not provide access to the full power
    of rsync, but does make most invocations easier to follow.

    Options (= is mandatory):

    - archive
          Mirrors the rsync archive flag, enables recursive, links,
          perms, times, owner, group flags and -D. (Choices: yes, no)

    - copy_links
          Copy symlinks as the item that they point to (the referent)
          is copied, rather than the symlink. (Choices: yes, no)

    - delete
          Delete files that don't exist (after transfer, not before)
          in the `src' path. (Choices: yes, no)

    = dest
          Path on the destination machine that will be synchronized
          from the source; The path can be absolute or relative.

    - dest_port
          Port number for ssh on the destination host. The
          ansible_ssh_port inventory var takes precedence over this
          value.

    - dirs
          Transfer directories without recursing (Choices: yes, no)

    - existing_only
          Skip creating new files on receiver. (Choices: yes, no)

    - group
          Preserve group (Choices: yes, no)

    - links
          Copy symlinks as symlinks. (Choices: yes, no)

    - mode
          Specify the direction of the synchroniztion. In push mode
          the localhost or delegate is the source; In pull mode the
          remote host in context is the source. (Choices: push, pull)

    - owner
          Preserve owner (super user only) (Choices: yes, no)

    - perms
          Preserve permissions. (Choices: yes, no)

    - recursive
          Recurse into directories. (Choices: yes, no)

    - rsync_path
          Specify the rsync command to run on the remote machine. See
          `--rsync-path' on the rsync man page.

    - rsync_timeout
          Specify a --timeout for the rsync command in seconds.

    = src
          Path on the source machine that will be synchronized to the
          destination; The path can be absolute or relative.

    - times
          Preserve modification times (Choices: yes, no)

    Notes:    Inspect the verbose output to validate the destination
          user/host/path are what was expected.The remote user for the
          dest path will always be the remote_user, not the
          sudo_user.Expect that dest=~/x will be ~<remote_user>/x even
          if using sudo.To exclude files and directories from being
          synchronized, you may add `.rsync-filter' files to the
          source directory.

    # Synchronization of src on the control machine to dest on the remote hosts
    synchronize: src=some/relative/path dest=/some/absolute/path
    
    # Synchronization without any --archive options enabled
    synchronize: src=some/relative/path dest=/some/absolute/path archive=no
    
    # Synchronization with --archive options enabled except for --recursive
    synchronize: src=some/relative/path dest=/some/absolute/path recursive=no
    
    # Synchronization without --archive options enabled except use --links
    synchronize: src=some/relative/path dest=/some/absolute/path archive=no links=yes
    
    # Synchronization of two paths both on the control machine
    local_action: synchronize src=some/relative/path dest=/some/absolute/path
    
    # Synchronization of src on the inventory host to the dest on the localhost in
    pull mode
    synchronize: mode=pull src=some/relative/path dest=/some/absolute/path
    
    # Synchronization of src on delegate host to dest on the current inventory host
    synchronize: >
        src=some/relative/path dest=/some/absolute/path
        delegate_to: delegate.host
    
    # Synchronize and delete files in dest on the remote host that are not found in src of localhost.
    synchronize: src=some/relative/path dest=/some/absolute/path delete=yes
    
    # Synchronize using an alternate rsync command
    synchronize: src=some/relative/path dest=/some/absolute/path rsync_path="sudo rsync"
    
    # Example .rsync-filter file in the source directory
    - var       # exclude any path whose last part is 'var'
    - /var      # exclude any path starting with 'var' starting at the source directory
    + /var/conf # include /var/conf even though it was previously excluded

> SYSCTL
=======================

::

    This module manipulates sysctl entries and optionally performs a
    `/sbin/sysctl -p' after changing them.

    Options (= is mandatory):

    - ignoreerrors
          Use this option to ignore errors about unknown keys.
          (Choices: yes, no)

    = name
          The dot-separated path (aka `key') specifying the sysctl
          variable.

    - reload
          If `yes', performs a `/sbin/sysctl -p' if the `sysctl_file'
          is updated. If `no', does not reload `sysctl' even if the
          `sysctl_file' is updated. (Choices: yes, no)

    - state
          Whether the entry should be present or absent in the sysctl
          file. (Choices: present, absent)

    - sysctl_file
          Specifies the absolute path to `sysctl.conf', if not
          `/etc/sysctl.conf'.

    - sysctl_set
          Verify token value with the sysctl command and set with -w
          if necessary (Choices: yes, no)

    - value
          Desired value of the sysctl key.

    # Set vm.swappiness to 5 in /etc/sysctl.conf
    - sysctl: name=vm.swappiness value=5 state=present
    
    # Remove kernel.panic entry from /etc/sysctl.conf
    - sysctl: name=kernel.panic state=absent sysctl_file=/etc/sysctl.conf
    
    # Set kernel.panic to 3 in /tmp/test_sysctl.conf
    - sysctl: name=kernel.panic value=3 sysctl_file=/tmp/test_sysctl.conf reload=no
    
    # Set ip fowarding on in /proc and do not reload the sysctl file
    - sysctl: name="net.ipv4.ip_forward" value=1 sysctl_set=yes
    
    # Set ip forwarding on in /proc and in the sysctl file and reload if necessary
    - sysctl: name="net.ipv4.ip_forward" value=1 sysctl_set=yes state=present reload=yes

> TEMPLATE
=======================

::

    Templates are processed by the Jinja2 templating language
    (http://jinja.pocoo.org/docs/) - documentation on the template
    formatting can be found in the Template Designer Documentation
    (http://jinja.pocoo.org/docs/templates/).Six additional variables
    can be used in templates: `ansible_managed' (configurable via the
    `defaults' section of `ansible.cfg') contains a string which can
    be used to describe the template name, host, modification time of
    the template file and the owner uid, `template_host' contains the
    node name of the template's machine, `template_uid' the owner,
    `template_path' the absolute path of the template,
    `template_fullpath' is the absolute path of the template, and
    `template_run_date' is the date that the template was rendered.
    Note that including a string that uses a date in the template will
    resort in the template being marked 'changed' each time.

    Options (= is mandatory):

    - backup
          Create a backup file including the timestamp information so
          you can get the original file back if you somehow clobbered
          it incorrectly. (Choices: yes, no)

    = dest
          Location to render the template to on the remote machine.

    - others
          all arguments accepted by the [file] module also work here,
          as well as the [copy] module (except the the 'content'
          parameter).

    = src
          Path of a Jinja2 formatted template on the local server.
          This can be a relative or absolute path.

    - validate
          validation to run before copying into place

    Notes:    Since Ansible version 0.9, templates are loaded with
          `trim_blocks=True'.Also, you can override jinja2 settings by
          adding a special header to template file. i.e.
          `#jinja2:variable_start_string:'[%' ,
          variable_end_string:'%]'' which changes the variable
          interpolation markers to  [% var %] instead of  {{ var }}.
          This is the best way to prevent evaluation of things that
          look like, but should not be Jinja2.  raw/endraw in Jinja2
          will not work as you expect because templates in Ansible are
          recursively evaluated.

    # Example from Ansible Playbooks
    - template: src=/mytemplates/foo.j2 dest=/etc/file.conf owner=bin group=wheel mode=0644
    
    # Copy a new "sudoers file into place, after passing validation with visudo
    - action: template src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'

> UNARCHIVE
=======================

::

    The [unarchive] module copies an archive file from the local
    machine to a remote and unpacks it.

    Options (= is mandatory):

    - copy
          Should the file be copied from the local to the remote
          machine? (Choices: yes, no)

    = dest
          Remote absolute path where the archive should be unpacked

    = src
          Local path to archive file to copy to the remote server; can
          be absolute or relative.

    Notes:    requires `tar'/`unzip' command on target hostcan handle `gzip',
          `bzip2' and `xz' compressed as well as uncompressed tar
          filesdetects type of archive automaticallyuses tar's `--diff
          arg' to calculate if changed or not. If this `arg' is not
          supported, it will always unpack the archivedoes not detect
          if a .zip file is different from destination - always
          unzipsexisting files/directories in the destination which
          are not in the archive are not touched.  This is the same
          behavior as a normal archive extractionexisting
          files/directories in the destination which are not in the
          archive are ignored for purposes of deciding if the archive
          should be unpacked or not

    # Example from Ansible Playbooks
    - unarchive: src=foo.tgz dest=/var/lib/foo

> URI
=======================

::

    Interacts with HTTP and HTTPS web services and supports Digest,
    Basic and WSSE HTTP authentication mechanisms.

    Options (= is mandatory):

    - HEADER_
          Any parameter starting with "HEADER_" is a sent with your
          request as a header. For example, HEADER_Content-
          Type="application/json" would send the header "Content-Type"
          along with your request with a value of "application/json".

    - body
          The body of the http request/response to the web service.

    - creates
          a filename, when it already exists, this step will not be
          run.

    - dest
          path of where to download the file to (if desired). If
          `dest' is a directory, the basename of the file on the
          remote server will be used.

    - follow_redirects
          Whether or not the URI module should follow redirects. `all'
          will follow all redirects. `safe' will follow only "safe"
          redirects, where "safe" means that the client is only doing
          a GET or HEAD on the URI to which it is being redirected.
          `none' will not follow any redirects. Note that `yes' and
          `no' choices are accepted for backwards compatibility, where
          `yes' is the equivalent of `all' and `no' is the equivalent
          of `safe'. `yes' and `no' are deprecated and will be removed
          in some future version of Ansible. (Choices: all, safe,
          none)

    - force_basic_auth
          httplib2, the library used by the uri module only sends
          authentication information when a webservice responds to an
          initial request with a 401 status. Since some basic auth
          services do not properly send a 401, logins will fail. This
          option forces the sending of the Basic authentication header
          upon initial request. (Choices: yes, no)

    - method
          The HTTP method of the request or response. (Choices: GET,
          POST, PUT, HEAD, DELETE, OPTIONS, PATCH)

    - others
          all arguments accepted by the [file] module also work here

    - password
          password for the module to use for Digest, Basic or WSSE
          authentication.

    - removes
          a filename, when it does not exist, this step will not be
          run.

    - return_content
          Whether or not to return the body of the request as a
          "content" key in the dictionary result. If the reported
          Content-type is "application/json", then the JSON is
          additionally loaded into a key called `json' in the
          dictionary results. (Choices: yes, no)

    - status_code
          A valid, numeric, HTTP status code that signifies success of
          the request.

    - timeout
          The socket level timeout in seconds

    = url
          HTTP or HTTPS URL in the form
          (http|https)://host.domain[:port]/path

    - user
          username for the module to use for Digest, Basic or WSSE
          authentication.

    Requirements:    urlparse, httplib2

    # Check that you can connect (GET) to a page and it returns a status 200
    - uri: url=http://www.example.com
    
    # Check that a page returns a status 200 and fail if the word AWESOME is not in the page contents.
    - action: uri url=http://www.example.com return_content=yes
      register: webpage
    
    - action: fail
      when: 'AWESOME' not in "{{ webpage.content }}"
    
    
    # Create a JIRA issue.
    - action: >
            uri url=https://your.jira.example.com/rest/api/2/issue/ 
            method=POST user=your_username password=your_pass 
            body="{{ lookup('file','issue.json') }}" force_basic_auth=yes 
            status_code=201 HEADER_Content-Type="application/json"  
    
    - action: > 
            uri url=https://your.form.based.auth.examle.com/index.php 
            method=POST body="name=your_username&password=your_password&enter=Sign%20in" 
            status_code=302 HEADER_Content-Type="application/x-www-form-urlencoded"
      register: login
    
    # Login to a form based webpage, then use the returned cookie to
    # access the app in later tasks.
    - action: uri url=https://your.form.based.auth.example.com/dashboard.php
                method=GET return_content=yes HEADER_Cookie="{{login.set_cookie}}"

> URPMI
=======================

::

    Manages packages with `urpmi' (such as for Mageia or Mandriva)

    Options (= is mandatory):

    - force
          Corresponds to the `--force' option for `urpmi'. (Choices:
          yes, no)

    - no-suggests
          Corresponds to the `--no-suggests' option for `urpmi'.
          (Choices: yes, no)

    = pkg
          name of package to install, upgrade or remove.

    - state
          Indicates the desired package state (Choices: absent,
          present)

    - update_cache
          update the package database first `urpmi.update -a'.
          (Choices: yes, no)

    # install package foo
    - urpmi: pkg=foo state=present
    # remove package foo
    - urpmi: pkg=foo state=absent
    # description: remove packages foo and bar 
    - urpmi: pkg=foo,bar state=absent
    # description: update the package database (urpmi.update -a -q) and install bar (bar will be the updated if a newer version exists) 
    - urpmi: name=bar, state=present, update_cache=yes     

> USER
=======================

::

    Manage user accounts and user attributes.

    Options (= is mandatory):

    - append
          If `yes', will only add groups, not set them to just the
          list in `groups'.

    - comment
          Optionally sets the description (aka `GECOS') of user
          account.

    - createhome
          Unless set to `no', a home directory will be made for the
          user when the account is created or if the home directory
          does not exist. (Choices: yes, no)

    - force
          When used with `state=absent', behavior is as with `userdel
          --force'. (Choices: yes, no)

    - generate_ssh_key
          Whether to generate a SSH key for the user in question. This
          will *not* overwrite an existing SSH key. (Choices: yes, no)

    - group
          Optionally sets the user's primary group (takes a group
          name).

    - groups
          Puts the user in this comma-delimited list of groups. When
          set to the empty string ('groups='), the user is removed
          from all groups except the primary group.

    - home
          Optionally set the user's home directory.

    - login_class
          Optionally sets the user's login class for FreeBSD, OpenBSD
          and NetBSD systems.

    - move_home
          If set to `yes' when used with `home=', attempt to move the
          user's home directory to the specified directory if it isn't
          there already. (Choices: yes, no)

    = name
          Name of the user to create, remove or modify.

    - non_unique
          Optionally when used with the -u option, this option allows
          to change the user ID to a non-unique value. (Choices: yes,
          no)

    - password
          Optionally set the user's password to this crypted value.
          See the user example in the github examples directory for
          what this looks like in a playbook. The `FAQ
          <http://docs.ansible.com/faq.html#how-do-i-generate-crypted-
          passwords-for-the-user-module>`_ contains details on various
          ways to generate these password values.

    - remove
          When used with `state=absent', behavior is as with `userdel
          --remove'. (Choices: yes, no)

    - shell
          Optionally set the user's shell.

    - ssh_key_bits
          Optionally specify number of bits in SSH key to create.

    - ssh_key_comment
          Optionally define the comment for the SSH key.

    - ssh_key_file
          Optionally specify the SSH key filename.

    - ssh_key_passphrase
          Set a passphrase for the SSH key.  If no passphrase is
          provided, the SSH key will default to having no passphrase.

    - ssh_key_type
          Optionally specify the type of SSH key to generate.
          Available SSH key types will depend on implementation
          present on target host.

    - state
          Whether the account should exist.  When `absent', removes
          the user account. (Choices: present, absent)

    - system
          When creating an account, setting this to `yes' makes the
          user a system account.  This setting cannot be changed on
          existing users. (Choices: yes, no)

    - uid
          Optionally sets the `UID' of the user.

    - update_password
          `always' will update passwords if they differ.  `on_create'
          will only set the password for newly created users.
          (Choices: always, on_create)

    Requirements:    useradd, userdel, usermod

    # Add the user 'johnd' with a specific uid and a primary group of 'admin'
    - user: name=johnd comment="John Doe" uid=1040
    
    # Remove the user 'johnd'
    - user: name=johnd state=absent remove=yes
    
    # Create a 2048-bit SSH key for user jsmith
    - user: name=jsmith generate_ssh_key=yes ssh_key_bits=2048

> VIRT
=======================

::

    Manages virtual machines supported by `libvirt'.

    Options (= is mandatory):

    - command
          in addition to state management, various non-idempotent
          commands are available. See examples (Choices: create,
          status, start, stop, pause, unpause, shutdown, undefine,
          destroy, get_xml, autostart, freemem, list_vms, info,
          nodeinfo, virttype, define)

    = name
          name of the guest VM being managed. Note that VM must be
          previously defined with xml.

    - state
          Note that there may be some lag for state requests like
          `shutdown' since these refer only to VM states. After
          starting a guest, it may not be immediately accessible.
          (Choices: running, shutdown)

    - uri
          libvirt connection uri

    - xml
          XML document used with the define command

    Requirements:    libvirt

    # a playbook task line:
    - virt: name=alpha state=running
    
    # /usr/bin/ansible invocations
    ansible host -m virt -a "name=alpha command=status"
    ansible host -m virt -a "name=alpha command=get_xml"
    ansible host -m virt -a "name=alpha command=create uri=lxc:///"
    
    # a playbook example of defining and launching an LXC guest
    tasks:
      - name: define vm
        virt: name=foo
              command=define
              xml="{{ lookup('template', 'container-template.xml.j2') }}"
              uri=lxc:///
      - name: start vm
        virt: name=foo state=running uri=lxc:///

> WAIT_FOR
=======================

::

    Waiting for a port to become available is useful for when services
    are not immediately available after their init scripts return -
    which is true of certain Java application servers. It is also
    useful when starting guests with the [virt] module and needing to
    pause until they are ready. This module can also be used to wait
    for a file to be available on the filesystem or with a regex match
    a string to be present in a file.

    Options (= is mandatory):

    - delay
          number of seconds to wait before starting to poll

    - host
          hostname or IP address to wait for

    - path
          path to a file on the filesytem that must exist before
          continuing

    - port
          port number to poll

    - search_regex
          with the path option can be used match a string in the file
          that must match before continuing.  Defaults to a multiline
          regex.

    - state
          either `present', `started', or `stopped'When checking a
          port `started' will ensure the port is open, `stopped' will
          check that it is closedWhen checking for a file or a search
          string `present' or `started' will ensure that the file or
          string is present before continuing (Choices: present,
          started, stopped)

    - timeout
          maximum number of seconds to wait for

    - connect_timeout
          
    
    # wait 300 seconds for port 8000 to become open on the host, don't start checking for 10 seconds
    - wait_for: port=8000 delay=10
    
    # wait until the file /tmp/foo is present before continuing
    - wait_for: path=/tmp/foo
    
    # wait until the string "completed" is in the file /tmp/foo before continuing
    - wait_for: path=/tmp/foo search_regex=completed
    

> XATTR
=======================

::

    Manages filesystem user defined extended attributes, requires that
    they are enabled on the target filesystem and that the
    setfattr/getfattr utilities are present.

    Options (= is mandatory):

    - follow
          if yes, dereferences symlinks and sets/gets attributes on
          symlink target, otherwise acts on symlink itself. (Choices:
          yes, no)

    - key
          The name of a specific Extended attribute key to
          set/retrieve

    = name
          The full path of the file/object to get the facts of

    - state
          defines which state you want to do. `read' retrieves the
          current value for a `key' (default) `present' sets `name' to
          `value', default if value is set `all' dumps all data `keys'
          retrieves all keys `absent' deletes the key (Choices: read,
          present, all, keys, absent)

    - value
          The value to set the named name/key to, it automatically
          sets the `state' to 'set'

    # Obtain the extended attributes  of /etc/foo.conf
    - xattr: name=/etc/foo.conf
    
    # Sets the key 'foo' to value 'bar'
    - xattr: path=/etc/foo.conf key=user.foo value=bar
    
    # Removes the key 'foo'
    - xattr: name=/etc/foo.conf key=user.foo state=absent

> YUM
=======================

::

    Installs, upgrade, removes, and lists packages and groups with the
    `yum' package manager.

    Options (= is mandatory):

    - conf_file
          The remote yum configuration file to use for the
          transaction.

    - disable_gpg_check
          Whether to disable the GPG checking of signatures of
          packages being installed. Has an effect only if state is
          `present' or `latest'. (Choices: yes, no)

    - disablerepo
          `repoid' of repositories to disable for the install/update
          operation These repos will not persist beyond the
          transaction Multiple repos separated with a ','

    - enablerepo
          Repoid of repositories to enable for the install/update
          operation. These repos will not persist beyond the
          transaction multiple repos separated with a ','

    - list
          Various (non-idempotent) commands for usage with
          `/usr/bin/ansible' and `not' playbooks. See examples.

    = name
          Package name, or package specifier with version, like
          `name-1.0'. When using state=latest, this can be '*' which
          means run: yum -y update. You can also pass a url or a local
          path to a rpm file.

    - state
          Whether to install (`present', `latest'), or remove
          (`absent') a package. (Choices: present, latest, absent)

    Requirements:    yum, rpm

    - name: install the latest version of Apache
      yum: name=httpd state=latest
    
    - name: remove the Apache package
      yum: name=httpd state=removed
    
    - name: install the latest version of Apche from the testing repo
      yum: name=httpd enablerepo=testing state=installed
    
    - name: upgrade all packages
      yum: name=* state=latest
    
    - name: install the nginx rpm from a remote repo
      yum: name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present
    
    - name: install nginx rpm from a local file
      yum: name=/usr/local/src/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present
    
    - name: install the 'Development tools' package group
      yum: name="@Development tools" state=present

> ZFS
=======================

::

    Manages ZFS file systems on Solaris and FreeBSD. Can manage file
    systems, volumes and snapshots. See zfs(1M) for more information
    about the properties.

    Options (= is mandatory):

    - aclinherit
          The aclinherit property. (Choices: discard, noallow,
          restricted, passthrough, passthrough-x)

    - aclmode
          The aclmode property. (Choices: discard, groupmask,
          passthrough)

    - atime
          The atime property. (Choices: on, off)

    - canmount
          The canmount property. (Choices: on, off, noauto)

    - casesensitivity
          The casesensitivity property. (Choices: sensitive,
          insensitive, mixed)

    - checksum
          The checksum property. (Choices: on, off, fletcher2,
          fletcher4, sha256)

    - compression
          The compression property. (Choices: on, off, lzjb, gzip,
          gzip-1, gzip-2, gzip-3, gzip-4, gzip-5, gzip-6, gzip-7,
          gzip-8, gzip-9, lz4, zle)

    - copies
          The copies property. (Choices: 1, 2, 3)

    - dedup
          The dedup property. (Choices: on, off)

    - devices
          The devices property. (Choices: on, off)

    - exec
          The exec property. (Choices: on, off)

    - jailed
          The jailed property. (Choices: on, off)

    - logbias
          The logbias property. (Choices: latency, throughput)

    - mountpoint
          The mountpoint property.

    = name
          File system, snapshot or volume name e.g. `rpool/myfs'

    - nbmand
          The nbmand property. (Choices: on, off)

    - normalization
          The normalization property. (Choices: none, formC, formD,
          formKC, formKD)

    - primarycache
          The primarycache property. (Choices: all, none, metadata)

    - quota
          The quota property.

    - readonly
          The readonly property. (Choices: on, off)

    - recordsize
          The recordsize property.

    - refquota
          The refquota property.

    - refreservation
          The refreservation property.

    - reservation
          The reservation property.

    - secondarycache
          The secondarycache property. (Choices: all, none, metadata)

    - setuid
          The setuid property. (Choices: on, off)

    - shareiscsi
          The shareiscsi property. (Choices: on, off)

    - sharenfs
          The sharenfs property.

    - sharesmb
          The sharesmb property.

    - snapdir
          The snapdir property. (Choices: hidden, visible)

    = state
          Whether to create (`present'), or remove (`absent') a file
          system, snapshot or volume. (Choices: present, absent)

    - sync
          The sync property. (Choices: on, off)

    - utf8only
          The utf8only property. (Choices: on, off)

    - volblocksize
          The volblocksize property.

    - volsize
          The volsize property.

    - vscan
          The vscan property. (Choices: on, off)

    - xattr
          The xattr property. (Choices: on, off)

    - zoned
          The zoned property. (Choices: on, off)

    # Create a new file system called myfs in pool rpool
    - zfs: name=rpool/myfs state=present
    
    # Create a new volume called myvol in pool rpool. 
    - zfs: name=rpool/myvol state=present volsize=10M
    
    # Create a snapshot of rpool/myfs file system.
    - zfs: name=rpool/myfs@mysnapshot state=present
    
    # Create a new file system called myfs2 with snapdir enabled
    - zfs: name=rpool/myfs2 state=present snapdir=enabled

> ZYPPER
=======================

::

    Manage packages on SuSE and openSuSE using the zypper and rpm
    tools.

    Options (= is mandatory):

    - disable_gpg_check
          Whether to disable to GPG signature checking of the package
          signature being installed. Has an effect only if state is
          `present' or `latest'. (Choices: yes, no)

    = name
          package name or package specifier wth version `name' or
          `name-1.0'.

    - state
          `present' will make sure the package is installed. `latest'
          will make sure the latest version of the package is
          installed. `absent'  will make sure the specified package is
          not installed. (Choices: present, latest, absent)

    Requirements:    zypper, rpm

    # Install "nmap"
    - zypper: name=nmap state=present
    
    # Remove the "nmap" package
    - zypper: name=nmap state=absent

> ZYPPER_REPOSITORY
=======================

::

    Add or remove Zypper repositories on SUSE and openSUSE

    Options (= is mandatory):

    - description
          A description of the repository

    - disable_gpg_check
          Whether to disable GPG signature checking of all packages.
          Has an effect only if state is `present'. (Choices: yes, no)

    = name
          A name for the repository.

    = repo
          URI of the repository or .repo file.

    - state
          A source string state. (Choices: absent, present)

    Requirements:    zypper

    # Add NVIDIA repository for graphics drivers
    - zypper_repository: name=nvidia-repo repo='ftp://download.nvidia.com/opensuse/12.2' state=present
    
    # Remove NVIDIA repository
    - zypper_repository: name=nvidia-repo repo='ftp://download.nvidia.com/opensuse/12.2' state=absent

