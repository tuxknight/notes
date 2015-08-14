.. _docker_in_action:

Docker 案例 -- 搭建工作流
==========================

流程
------

#. 在本地功能分支上完成应用代码

#. 在Github上发起一个到master分支的pull request

#. 在Docker容器上运行自动测试

#. 如果测试通过，手动将pull request merge进master分支

#. 一旦merge成功，再次运行自动测试

#. 如果第二次测试也通过，就在Docker Hub上对应用进行构建

#. 一旦构建完成，自动化的部署到生产环境

概念
------

* `Dockerfile <http://docs.docker.com/reference/builder/>`_ 中包含了一系列语句，用于对镜像的行为进行描述。
* `镜像 <http://docs.docker.com/terms/image/>`_ 是一个模板，用来保存环境状态并创建容器。
* `容器 <http://docs.docker.com/terms/container/>`_ 可以理解为实例化的镜像，并会在其中运行一系列进程。

Why ?
------

  使用Docker意味着能够在开发机上完美地模拟生产环境，而不用再为任何由两者环境、配置差异所造成的问题而担心，此外，docker带给我们的还有:

  * 良好的版本控制
  
  * 随时便捷地发布或重建整个开发环境
  
  * 一次构建，随处运行

配置Docker
------------


  * 由于windows NT、Darwin内核缺少运行Docker容器的一些Linux内核功能，需要借助boot2docker，一个用于运行Docker的轻量级Linux发行版。

  * Linux内核的操作系统可以直接运行docker守护进程。

  * ``docker version`` 

Compose UP
------------

  Docker compose 是官方提供的容器业务流程框架，只需通过简单的YAML配置文件就能完成多个容器服务的构建和运行。

  ``pip install docker-compose`` 安装docker compose

  开始搭建Flask+Redis应用

  在项目根目录下新建docker-compose.yml文件::

    web:
      build: web
    volumes:
      - web: /code
    ports:
      - "80:5000"
    links:
      - redis
    command: python app.py
    redis:
    image: redis:2.8.19
    ports:
      - "6379:6379"


  上面我们对项目所含的两个服务进行了操作:

    * web: 我们将在web目录下进行容器的构建，并且将其作为Volume挂载到容器的/code目录中，然后通过python app.py来启动Flask应用。最后将容器的5000端口暴露出来，映射到80端口上。

    * redis: 直接使用Docker Hub上的官方镜像来提供所需的Redis服务支持，将6379端口暴露并映射到主机上。

    之后在web目录下创建Dockerfile文件用于指导如何构建应用镜像。

构建并运行
-----------

  ``docker-compose up`` 这会根据dockerfile构建Flask应用的镜像，从官方仓库拉取Redis镜像，然后将一切运行起来。docker compose会并行地期都过全部容器，每个容器会被分配各自的名字。

  ``docker-compose ps`` 可以查看应用进程的运行状态。两个进程运行在不同的容器中，而Docker Compose将它们组织在一起。

  我们建立了本地环境，通过Dockerfile详尽描述了如何构建镜像，并基于该镜像启动了相应容器。我们使用Docker Compose来将这一切整合起来，包括构建和容器之间的关联、通信（在Flask和Redis进程之间）。


