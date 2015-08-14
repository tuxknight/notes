.. _install:

======================
Ansible quick start
======================

install
==========

  ::

    yum install ansible # RHEL/CentOS/Fedora

    apt-get install ansible # Debian/Ubuntu

    emerge -avt ansible  # Gentoo/Funtoo

    pip install ansible # will also install  paramiko PyYAML jinja2

Inventory
===========

  Inventory文件用来定义需要管理的主机，默认位置在 ``/etc/ansible/hosts`` , 使用 ``-i`` 选项指定非默认位置。

  被管理的机器通过IP或域名指定，未分组的机器保留在hosts文件的顶部，分组使用 ``[]`` 指定，同时支持嵌套::

    [miss]
    dn0[1:3]

    [lvs:children]
    web
    db

  在配置中可以为主机设定端口、默认用户、私钥等参数::

    [db]
    orcl01.example.com ansible_ssh_user=oracle

Try it out
=============

  ``ansible -i hosts all -m ping -u root -f 3`` 

  ::

    *. -i 指定了inventory文件，当前目录下的hosts文件，不指定则使用默认位置。

    *. all 针对hosts定义的所有主机执行，可以指定组名或者模式

    *. -m 指定所使用的模块，不指定则使用默认模块 command

    *. -u 指定远端机器的用户

    *. -a 指定传给模块的参数，Ansible以键值对(Key-Value)的方式接受参数，并以JSON格式返回结果。

    *. -f 同一时间只在指定数量的机器上执行

Ad-Hoc 简单任务
==================

执行命令
------------

  ``ansible -m raw -a 'yum install -y python-simplejson'``

  可以在对方机器上python版本为2.4或者其他没有装python的设备上使用raw模块执行命令。类似于直接在远端执行 shell命令。

  ``ansible-doc raw``

传输文件
-----------

  ``ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"``


模块
=======

setup
----------

``ansible -i hosts -m setup -a 'filter="ansible_processor_count"'`` 
可以获取主机的系统信息，并将其保存在内置变量中方便其他模块进行引用。

user
-----

``ansible -i hosts -m user -a 'name="test" shell="/bin/false"'``

file
------

``ansible -i hosts -m file -a 'path=/etc/fstab'``

``ansible -i hosts -m file -a 'path=/tmp/ksops state=directory mode=0755 owner=nobody'``

``ansible -i hosts -m file -a 'path=/tmp/file state=absent'``

copy
------

``ansible -i hosts -m copy -a 'src=vimrc dest=/root/.vimrc mode=644 owner=root'``

command
---------

``ansible -i hosts -m command -a 'rm -rfv /tmp/test removes=/tmp/test'``

``ansible -i hosts -m command -a 'touch /tmp/test creates=/tmp/test'``

使用command模块无法通过返回值判断命令是否执行成功，如果定义了creates属性，当文件存在时，不执行创建文件的命令，如果定义了removes属性，当文件不存在时，不执行删除文件的命令。

shell
-------

``ansible -i hosts -m shell -a '/home/monitor.sh > /home/applog/test.log creates=/home/applog/test.log'``

shell模块提供了对重定向、管道、后台任务等特性的支持。

Playbook 复杂任务
===================

  创建新用户::

    cat user.yml

    ---
    - name:create user  #Playbook名称
      hosts:db    #起作用的主机组
      user:root   #以指定用户执行
      gather_facts:false  #收集远端机器的相关信息

      vars:     #定义变量 user
      - user: "toy"
      tasks:    #指定要执行的任务
      - name:create {{user}} on miss
        user:name="{{user}}"
