.. _docker_compose:

Docker compose
=================

.. contents:: Topics

docker-compose.yml reference
---------------------------------

image
``````

tag or partial image ID. Can be local or remote. Compose will attempt to pull if it doesnot exist locally::

  image: ubuntu
  image: a4c42

build
```````
path to a directory containing a Dockerfile. When the value supplied is a relative path, it is interpreted as relative to the location of the yml file itself. This directory is also the build context that is sent to the Docker daemon.

Compose will build and tag it with a generated name and use that image thereafter

dockerfile
````````````
Alternate Dockerfile

compose will use an alternate file to build with

::

  dockerfile: Dockerfile-alternate

command
`````````

override the default command

:: 

  command: bundle exec thin -p 3000

links
``````

link to containers in another service. Either specify both the service name and the link alias (SERVICE:ALIAS), or just the service name::

  links:
    - db
    - db:database
    - redis

An entry with the alias' name will be created in ``/etc/hosts`` inside containers for this services::

  172.17.2.186 db
  172.17.2.186 database
  172.17.2.187 redis

external_links
`````````````````

link to containers started outside this ``docker-compose.yml`` or even outside of compose::

  external_links:
    - redis_1
    - project_db_1:mysql
    - project_db_1:postgresql

extra_hosts
`````````````

add hostname mapping::

  extra_hosts:
    - "host01:1.2.3.4"
    - "host02:2.2.3.4"

ports
```````

expose ports. Either specify both ports(HOSTPORT:CONTAINPORT) or just the container port::

  ports:
    - "3000"
    - "5601:5601"
    - "39200:9200"
    - "127.0.0.1:9200:9200"

.. note::

   When mapping ports in the HOST:CONTAINER format, you may experience erroneous results when using a container port lower than 60, because YAML will parse numbers in the format xx:yy as sexagesimal (base 60). For this reason, we recommend always explicitly specifying your port mappings as strings.

expose
````````

Expose ports without publishing them to the host machine - they will only be accessible to linked services::

  expose:
    - "6379"
    - "9200"

volumes
`````````

mount paths as volumes, optionally specifying a path on the host machine(HOST:CONTAINER) or an access mode(HOST:CONTAINER:ro)::

  volumes:
    - /var/lib/mysql
    - ~/configs:/etc/configs/:ro

volumes_from
`````````````

mount all of the volumes from another services or container::

  volues_from:
    - service_name
    - container_name

environment
````````````

add environment variables. You can use either an array or a dictionary.

Environment variables with only a key are resolved to their values on the machine Compose is running on, which can be helpful for secret or host-specific values::

  environment:
    RACK_ENV: development
    SESSION_SECRET:
  
  environment:
    - RACK_ENV=development
    - SESSION_SECRET

env_file
``````````

Add environment variables from a file. Can be a single value or a list.

If you have specified a Compose file with docker-compose -f FILE, paths in env_file are relative to the directory that file is in.

extends
`````````

Extend another service, in the current file or another, optionally overriding configuration.

Compose CLI reference
------------------------

build
```````

builds or rebuilds services

kill
``````

forces running contenters to stop by sending a ``SIGKILL`` signal. Optionally the signal can be passed::

  docker-compose kill -s SIGTERM

logs
``````

displays log output from services.

port
``````

prints the public port for a port binding

ps
```

lists containers

pull
`````

pulls service images

restart
`````````

restart services

rm
```

removes stopped service containers

run
````

run a one-off command on a service

::

  docker-compose run web python manage.py shell

In this example, compose will start ``web`` service then run ``manage.py shell`` in python. Note that by default, linked services will also be started, unless they are already running.

scale
``````

sets the number of containers to run for a service

::

  docker-compose scale web=2 worker=3

start
``````

starts existing containers for a service

stop
`````
stops running containers without removing them. They can be started again with ``docker-compose start``

up
```

builds, creates, starts, and attaches to containers for a service.

Linked services will be started, unless they are already running.

By default, ``docker-compose up`` will aggregate the output of each container and when it exits, all containers will be stopped.

Running ``docker-compose up -d`` will start the containers in the background and leave them running.

By default, ``docker-compose up`` will stop and recreate existing containers. If you do not want containers stopped and recreated, use ``docker-compose up --no-recreate`` . This will still start any stopped containers, if needed.

Options
`````````

--verbose

  shows more output

-v, --version
  prints version and exits

-f, --file FILE

  specify what files to read configuration from. If not provided, compose will look for docker-compose.yml

-p, --project-name NAME

  specifies an alternate project name( default: current directory name)

Environment Variables
```````````````````````

COMPOSE_PROJECT_NAME

  Sets the project name, which is prepended to the name of every container started by Compose

COMPOSE_FILE

  Specify what file to read configuration from. If not provided, Compose will look for docker-compose.yml in the current working directory, and then each parent directory successively, until found.

DOCKER_HOST

  Sets the URL of the docker daemon. As with the Docker client, defaults to unix:///var/run/docker.sock.

DOCKER_TLS_VERIFY

  When set to anything other than an empty string, enables TLS communication with the daemon.

DOCKER_CERT_PATH

  Configures the path to the ca.pem, cert.pem, and key.pem files used for TLS verification. Defaults to ~/.docker.
