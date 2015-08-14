.. _docker_command:

Docker入门 -- 命令
==================


.. _docker_info:

docker info
-------------

  通过运行 ``docker info`` 检测docker安装是否正确

  ::

    # docker info
    Containers: 16
    Images: 3
    Storage Driver: aufs
     Root Dir: /var/lib/docker/aufs
      Dirs: 35
      Execution Driver: native-0.2
      Kernel Version: 3.13.0-36-generic
      Operating System: Ubuntu 14.04.1 LTS
      CPUs: 4
      Total Memory: 3.729 GiB
      Name: nn
      ID: NZE2:4R7O:GGXU:WYP3:PC4W:3G23:CSM4:XIRH:RAIB:GRV2:HKS2:Y47U
      WARNING: No swap limit support

.. _docker_pull:

docker pull
--------------

  ``docker pull busybox`` 从Docker Hub上拉取busybox镜像

.. _docker_run:

docker run
-----------

  ``docker run busybox /bin/echo Hello Docker`` 运行容器

  ``docker run -d busybox /bin/sh -c "while true; do echo Docker; sleep 1; done"`` 以后台进程的方式运行容器

.. _docker_logs:

docker logs
------------
  
  ::

    # docker run -d busybox /bin/sh -c "while true; do echo Docker; sleep 1; done"
    adc090ccbcf7feb6c9cee7228cc32f2739eddfa78ad2b10a192d8e12b2a2c594

  在后台运行容器，会返回一个容器ID，之后使用 ``docker logs adc090ccbcf7feb6c9cee7228cc32f2739eddfa78ad2b10a192d8e12b2a2c594`` 可以查看输出的结果。容器ID长度很长，实际使用中可以只取前八位来使用。

.. _docker_stop:

docker start & stop & restart
--------------------------------

  ``docker stop docker_id`` 停止容器， ``docker restart docker_id`` 重启该容器。如果要完全移除容器，需要先停止容器，然后才能移除::

    docker stop acd090ccbc
    docker rm acd090ccbc

.. _docker_commit:

docker commit
---------------

  ``docker commit acd090ccbc sample01`` 将容器的状态保存为镜像，镜像名称为sample01,镜像名称只能取字符[a-z]和数字[0-9]

.. _docker_images:

docker images
-------------

  ``docker images`` 查看所有镜像的列表

.. _docker_search:

docker search
--------------

  ``docker search (image_name)`` 在Docker registry搜索镜像

.. _docker_history:

docker history
---------------

  ``docker history (image_name)`` 查看镜像的历史版本

.. _docker_push:

docker push
------------

  ``docker push (image_name)`` 将镜像推送到registry

.. _docker_build:

docker build
---------------

  ``docker build [options] PATH | URL`` 使用Dockerfile构建镜像

  ``--rm=true`` -- 构建成功后，移除所有中间容器

  ``--no-cache=false`` -- 构建过程中不使用缓存

.. _docker_attach:

docker attach
---------------

  ``docker attach container`` 附加到正在运行的容器上

.. _docker_diff:

docker diff
------------

  ``docker diff container`` 列出容器内发生变化的文件和目录

.. _docker_events:

docker events
--------------

  打印指定时间内容器的实时系统事件

.. _docker_import:

docker import
---------------

  导入远程文件、本地文件和目录::

    docker import http://example.com/example.tar
    tar -C image.tar | docker import - image_app

.. _docker_export:

docker export
---------------

  导出容器的系统文件打包成tarball::

    docker export container > imags.tar

.. _docker_cp:

docker cp
----------

  从容器内复制文件到指定路径上::

    docker cp container:path hostpath

.. _docker_login:

docker login
--------------

  用来登陆Docker Registry服务器::

    docker login [options] [server]
    docker login localhost:8080

.. _docker_inspect:

docker inspect
---------------

  收集关于容器和镜像的底层信息，包括::

    * 容器实例的IP地址

    * 端口绑定列表

    * 特定端口映射的搜索

    * 收集配置的详细信息

  ``docker inspect [ --format= ] container/image``

  ``docker inispact --format='{{.State}}' container/image``

.. _docker_kill:

docker kill
------------

  发送 ``SIGKILL`` 信号来停止容器的主进程::

    docker kill [options] container

.. _docker_rmi:

docker rmi
-----------

  移除一个或多个镜像::

    docker rmi image

.. _docker_wait:

docker wait
-------------

  阻塞对指定容器的其他调用方法，直到容器停止后退出阻塞。

.. _docker_load:

docker load
------------

  从tarball中载入镜像到STDIN::

    docker load -i app_box.tar

.. _docker_save:

docker save
------------

  将镜像保存为tarball并发送到STDOUT::

    docker save image > app_box.tar
