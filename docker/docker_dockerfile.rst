.. _docker_dockerfile:

Dockerfile
===========

  Dockerfile包含创建镜像所需要的全部指令。基于在Dockerfile中的指令，我们可以使用Docker build命令来创建镜像。通过减少镜像和容器的创建过程来简化部署。

  Dockerfile支持的语法命令如下::
   
    INSTRUCTION argument

  指令不区分大小写，但是命名约定为全部大写。

FROM
------

  所有Dockerfile必须以 ``FROM`` 命令开始。 ``FROM`` 命令指定镜像基于哪个基础镜像创建。接下来的命令也会基于这个基础镜像。可以使用多次FROM命令，表示会创建多个镜像。

  ::

    FROM <image>
    FROM <image>:<tag>
    FROM <image>@<digest>

  例如 ``FROM ubuntu`` 这样的指令告诉我们，新的镜像将基于Ubuntu的景象来构建。

MAINTAINER
-----------

  设置该镜像的作者::

    MAINTAINER <author name>

RUN
----

  在shell或exec环境下执行的命令。会在新创建的镜像上添加新的层面，接下来提交的镜像用在Dockerfile的下一条指令中。

  ::

    RUN <command>   shell form
    RUN ["executable", "param1", "param2"]  exec form

  exec形式没有变量扩展，如 ``RUN ["echo", "$HOME"]`` 不会展开 $HOME,需要改成 ``RUN ["sh", "-c", "echo", "$HOME"]``

ADD
----

  复制文件指令，将URL或者启动配置上下文中的一个文件复制到容器内的位置::

    ADD <src> <destination>

CMD
----

  提供了容器默认的执行命令。Dockerfile只允许使用一次CMD指令，使用多个CMD会抵消之前所有的指令，只有最后一个指令失效::

    CMD ["executable", "param1", "param2"]

    CMD ["param1", "param2"]

    CMD command param1 param2

.. note::

  CMD ["executable", "param1", "param2"] 是推荐使用的格式。exec form被解析成JSON数组，因此不能使用单引号，要使用双引号。

EXPOSE
-------

  指定容器在运行时监听的端口

  ::

    EXPOSE <port>

ENTRYPOINT
-----------

  配置给容器一个可执行的命令，这意味着每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序。类似与CMD，Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令。

  ::

    ENTRYPOINT ["executable", "param1", "param2"]

    ENTRYPOINT command param1 param2

WORKDIR
--------

  指定RUN、CMD和ENTRYPOINT命令的工作目录。

  ::

    WORKDIR /path/to/workdir

ENV
----

  设置环境变量，使用键值对，增加运行程序的灵活性。

  ::

    ENV <key> <value>

USER
-----

  镜像正在运行时设置一个UID。

  ::

    USER <uid>

VOLUME
-------

  授权容器访问主机上的目录。

  ::

    VOLUME ["/data"]


example
--------

::

  # Nginx
  #
  # VERSION               0.0.1
  
  FROM      ubuntu
  MAINTAINER Victor Vieux <victor@docker.com>
  
  LABEL Description="This image is used to start the foobar executable" Vendor="ACME Products" Version="1.0"
  RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
  
  # Firefox over VNC
  #
  # VERSION               0.3
  
  FROM ubuntu
  
  # Install vnc, xvfb in order to create a 'fake' display and firefox
  RUN apt-get update && apt-get install -y x11vnc xvfb firefox
  RUN mkdir ~/.vnc
  # Setup a password
  RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
  # Autostart firefox (might not be the best way, but it does the trick)
  RUN bash -c 'echo "firefox" >> /.bashrc'
  
  EXPOSE 5900
  CMD    ["x11vnc", "-forever", "-usepw", "-create"]
  
  # Multiple images example
  #
  # VERSION               0.1
  
  FROM ubuntu
  RUN echo foo > bar
  # Will output something like ===> 907ad6c2736f
  
  FROM ubuntu
  RUN echo moo > oink
  # Will output something like ===> 695d7793cbe4
  
  # You᾿ll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
  # /oink.


.. seealso::

  #. :ref:`Dockerfile best practices <dockerfile_best_practices_take>`
