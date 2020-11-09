
Docker
===============
1. __Intro__
1. __CLI__
1. __Buildear __
1. __Multiple Containers__
1. __Docker Swarm__

Part 1
=========================

## Intro

Que es Docker
================

Conceptos claves
----------------
* Entorno definido para la ejecucion de procesos
* Aislamientos entre entornos
* Es un sandbox con recursos definidos
* Permite una interfaz sencilla para lanzar procesos
* Utiliza un sistema de filesystem en capas (Layers) entonces los containers que comparten layers no ocupan mas espacio hasta que modifico esa layer.

Que NO es docker
--------------------
* No es virtualizacion
* No usa un kernel aparte
* No es un hypervisor

Installation
===============
Docker esta basado en linux pero se puede instalar en Windows y Mac OS X

Instrucciones: <https://docs.docker.com/installation/>

Para Ubuntu/Debian :

    curl https://get.docker.com/ | sh

Part 2
=========================

## CLI o Command line interface

Docker hello world
=========================

    docker run busybox echo 'Hello World'

What has happened?

* Descarga `busybox` que contiene un par de utilidades practicas de Linux
* Crea un nuevo container
* Ejecuta el comando `echo` dentro del nuevo container

Imagenes y Containers
==========================

Docker Image
---------------

* Es una plantilla inmutable para los containers
* Puede hacerse push y pull de un registry
* Las imagenes tienen la siguiente sintaxis `[registry/][user/]name[:tag]`
* El tag por defecto es `latest`

Docker Container
---------------

* Es una instancia de una imagen (plantilla)
* Puedo arrancarla detenerla o reiniciarla (start stop restart)
* Persiste cambios dentro de su filesystem


Commands for image handling
==============================

search, pull & push
----------------------

Buscar en el registry:

    docker search <term>

Descargar del registry:

    docker pull <image>

Subir al registry:

    docker push <image>

Comandos para el manejo de imagenes
==============================

list, tag & delete
----------------------

Listar imagenes disponibles:

    docker images

Darle un nombre a una imagen (alias):

    docker tag <oldname> <newname>

borrar una imagen:

    docker rmi <image>

Docker run
===============

Start a new container

    docker run <nombredeimagen>

Opciones:

    Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
     --name             Give the container a symbolic name
     -v, --volume=[]    Bind mount a volume
     -p, --publish=[]   Publish a container's port(s) to the host
     -e, --env=[]       Set environment variables
     --link=[]          Add link to another container
     --restart="no"     Restart policy (no, on-failure[:max-retry], always)
     --rm=false         Automatically remove the container when it exits
     -d, --detach=false Run container in background and print container ID
     -i, --interactive=false   Keep STDIN open even if not attached
     -t, --tty=false    Allocate a pseudo-TTY

Some option details
====================

* Publicar el puerto 80 del container como el 8080 en el host: `-p 8080:80`
* Montar un directorio local `/html` como el directorio `/usr/share/nginx/html`
  en el container: `-v /html:/usr/share/nginx/html`
  * Con “Mount” nos referimos a hacerlo disponible.
  * `/usr/share/nginx/html` Aca es donde NGINX espera los archivos HTML.

Exercise:
----------
1. Arrancar un servidor Nginx en el puerto 8080 del host,
   con el archivo `index.html` por defecto.
   * Ir a `http://localhost:8080/` ver que ande.
1. Arrancar un servidor Nginx en el puerto 8080 del host con un archivo
  `index.html` personalizado.
   * Ir a `http://localhost:8080/` ver que ande.

Ver los containers:
========================

Listar containers
-----------------
Containers ejecutandose:

    docker ps

Todos los containers:

    docker ps -a

Ciclo de vida de un container
=========================

STOP:

    docker stop <container..>

Start:

    docker start <container..>

Kill:

    docker kill <container..>

Remove:

    docker rm <container..>


Debugging
==========================

exec
-----

Ejecuta un comando dentro de la consola del container (sh o bash)

    docker exec <container> <command>
    docker exec -it <container> bash

logs
--------
Ver los logs (stdout) del container container.

    docker logs -f <container>

copy
-------
copy archivos desde y hacia el container, e.g.

    docker cp my_webserver:/etc/nginx/nginx.conf ~/

Interaction and debugging
==========================

Ejercicio
----------
1. Arrancar un webserver
2. Reemplazar el archivo index.html
3. Ver los logs del webserver


Modify a container
==========================

Exercise
----------
1. Search online for the Docker registry image
1. Start a private Docker image registry
    - Search for the image at <https://hub.docker.com/>
    - Start the image locally
1. Commit your container’s changes from the previous exercise
1. Push your new image to your private registry
1. Delete the image locally
1. Start the image again (now coming from the registry)

Useful tricks: Cleanup Script
=======================================
You have to cleanup your local images and old containers regularly.

    docker rm $(docker ps -q -a -f status=exited)
    docker rmi $(docker images -q -f dangling=true)


Especially on test and build systems this should be part of a cron job.


    exited=$(docker ps -q -a -f status=exited | wc -l)

    if [ "$exited" != "0" ]; then
            docker rm $(docker ps -q -a -f status=exited)
    fi

    tagref=$(docker images -q -f dangling=true | wc -l)

    if [ "$tagref" != "0" ]; then0
            docker rmi $(docker images -q -f dangling=true)
    fi

Part 3
=========================

## Build Docker Images

docker build
=====================

The normal way to create images is through `Dockerfile` build descriptions.

1. create a `Dockerfile`, e.g.

        FROM nginx
        COPY index.html /usr/share/nginx/html/

2. build the image and give it a name

        docker build --pull -t my-nginx .


Note:
---------------------
- The build has the current directory as context
- All paths are relative to the Dockerfile
- Each command in the Dockerfile creates a new (temporary container)
- Every creation step is cached, so repeated builds are fast


Dockerfile
=====================

FROM
--------

The `FROM` instruction sets the Base Image:

    FROM <image>:<tag>

Example:

    FROM nginx:15:04

Dockerfile
=====================

COPY
--------
`COPY` can be used to copy files or directories to the image.

    COPY <src>... <dest>

- Source can contain wildcards
- If dest does not exist it will be created

Example:

    COPY service.config /etc/service/
    COPY service.config /etc/service/myconfig.cfg
    COPY *.config /etc/service/
    COPY cfg/ /etc/service/

Dockerfile
=====================

Exercise
----------

Recreate your webserver image with static content using `docker build`

Dockerfile
=====================

CMD
--------

With `CMD` you can specify the default command to execute on container startup.
It has two forms.

The exec form (preferred):

    CMD ["executable","param1","param2"]

The shell form:

    CMD executable param1 param2

__Attention__: the shell form starts the command in a sub process, so it will not get
the process id 1 and will not receive all signals e.g. from command line or `docker stop`.

Example:

    CMD ["nginx", "-g", "daemon off;"]

Dockerfile
=====================

ENTRYPOINT
-----------

The command in `ENTRYPOINT` will be executed on startup and allows you to configure a container that will run as an executable.

- The arguments in `CMD` are passed to the entrypoint by default
- If supplied, the `docker run` arguments overwrite those of the `CMD` and are passed as entrypoint arguments.

The exec form (preferred):

    ENTRYPOINT ["executable", "param1", "param2"]

Example:

    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

Dockerfile
=====================

RUN
--------

The `RUN` command allows to execute arbitrary commands in the container, which modify the
file system and produce a new layered container.

    RUN <command>

It is common to tie related commands together into one RUN command, using shell magic.

Example:

    RUN apt-get update && \
        apt-get install -y ca-certificates nginx=${NGINX_VERSION} && \
        rm -rf /var/lib/apt/lists/*

Dockerfile
=====================

ENV
------------------------
`ENV` sets environment variables which are present during container build and remain existent in the image.

    ENV <key> <value>
    ENV <key>=<value> ...

On container startup they can be overwritten with the `-e` or `--env` option:

    docker run -e key=value my_image

Example:

    docker run -e message='The answer is' -e answer=42 \
        ubuntu \
        bash -c 'echo $message $answer'
    The answer is 42

Dockerfile
==========================

Exercise
----------
Create a `greeting` image which can echo a configurable hello world greeting message in ASCII art (e.g. using the ubuntu package figlet):

    docker run --rm greeting
    >  _   _       _ _         __  __                  _
    > | | | | __ _| | | ___   |  \/  | __ _ _ ____   _(_)_ __
    > | |_| |/ _` | | |/ _ \  | |\/| |/ _` | '__\ \ / / | '_ \
    > |  _  | (_| | | | (_) | | |  | | (_| | |   \ V /| | | | |
    > |_| |_|\__,_|_|_|\___/  |_|  |_|\__,_|_|    \_/ |_|_| |_|

    docker run --rm -e message=Hi greeting Arthur
    >  _   _ _      _         _   _
    > | | | (_)    / \   _ __| |_| |__  _   _ _ __
    > | |_| | |   / _ \ | '__| __| '_ \| | | | '__|
    > |  _  | |  / ___ \| |  | |_| | | | |_| | |
    > |_| |_|_| /_/   \_\_|   \__|_| |_|\__,_|_|


Dockerfile
=====================

ADD
--------
`ADD` can do the same as `COPY` with the following additions:

- If src is an URL, the file is downloaded
- If src is a local tar archive, it will be extracted to dest

Example:

    ADD https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.4.tar.gz /es/
    RUN cd /es && tar xvfz elasticsearch-1.4.4.tar.gz

    ADD configs.tar.gz /etc/service/

Dockerfile
=====================

VOLUME
------------------------
Declare folders for volume mounts.

    VOLUME ["/data"]

Benefit:
-------
- The user of your image has explicit documentation of the available mounts
- The Docker daemon and cloud tools can persist and backup them
- You can use the volumes from other containers by

        docker run --volumes-from container_with_volumes

Dockerfile
=====================

EXPOSE
------------------------
With `EXPOSE` an image can declare the ports which should be exported.

    EXPOSE <port> [<port>...]

Benefit:
-------
- This information is needed for communication between linked containers
- The exposed ports can be used by the `docker run -P`:

        -P, --publish-all=false  Publish all exposed ports to random ports


Example nginx
=================
    FROM debian:jessie

    MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"

    RUN apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
    RUN echo "deb http://nginx.org/packages/mainline/debian/ jessie nginx" >> /etc/apt/sources.list

    ENV NGINX_VERSION 1.9.3-1~jessie

    RUN apt-get update && \
        apt-get install -y ca-certificates nginx=${NGINX_VERSION} && \
        rm -rf /var/lib/apt/lists/*

    # forward request and error logs to docker log collector
    RUN ln -sf /dev/stdout /var/log/nginx/access.log
    RUN ln -sf /dev/stderr /var/log/nginx/error.log

    VOLUME ["/var/cache/nginx"]

    EXPOSE 80 443

    CMD ["nginx", "-g", "daemon off;"]


Part 4
=========================

## Multiple Containers

Multiple containers
==========================

The power of Docker comes in, when you compose your apps out of multiple containers.

- Networking
- docker-compose

Networking Types
=========================

By default Docker comes with the following networks:

    docker network ls

    NETWORK ID          NAME                DRIVER
    7fca4eb8c647        bridge              bridge
    9f904ee27bf5        none                null
    cf03ee007fb4        host                host

Docker can manage networks of different types:

* __host__: The host interface
* __bridge__: Bridged network interfaces
* __overlay__: Software defined multi host network (swarm only)

Manage networks with:

    docker network create|rm|inspect|ls

Connect Networks
=========================

Containers can be connected to multiple networks.

Network at startup:

    docker run --net=<networkname> <image>

Connect a running container:

    docker network connect <networkname> <containerid>

Disconnect a running container:

    docker network disconnect <networkname> <containerid>


docker-compose
================
docker-compose is a simple tool to start multiple containers.

Installation:
----------------

    curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-`uname -s`-`uname -m` > docker-compose
    chmod a+x docker-compose

Configuration by `docker-compose.yml`:

    version: '2'
    services:
      web:
        build: .
        ports:
          - "5000:5000"
        volumes:
          - .:/code

      redis:
        image: redis


docker-compose usage:
=====================
    Usage:
      docker-compose [options] [COMMAND] [ARGS...]
      docker-compose -h|--help

    Commands:
      up                 Create and start containers
      down               Stop and remove containers, networks, images, and volumes
      build              Build or rebuild services
      config             Validate and view the compose file
      events             Receive real time events from containers
      exec               Execute a command in a running container
      logs               View output from containers
      ps                 List containers
      pull               Pull service images
      push               Push service images
      restart            Restart services
      rm                 Remove stopped containers
      scale              Set number of containers for a service
      start              Start services
      stop               Stop services
      .. there are some more ..

Exercise
===================
1. Setup a docker-compose project with:
   - webserver with php
   - database of choice (e.g. mysql, postgres, or nosql, ...)
2. Implement a counter example in php
3. Scale the webserver

compose best practices
==============================
* Environment variables can be used
* Additional environment variables can be defined in an environment file: `.env`
* Image tags can be defined by variables
* Compose files can be extended
  * Use one base file
  * One extension per environment

Part 5
=========================

## Docker Swarm

Docker Swarm
=======================
Swarm is a build in Docker clustering mode.
It provides:

* Single master for communication with the cluster
* Monitoring and failover between node
* Scaling of containers
* Load balancing of published ports


Setup a swarm
================
Swarm setup with 3 virtualbox nodes

Requirements:

* Oracle Virtualbox has to be installed → https://www.virtualbox.org/wiki/Downloads
* Docker >= 1.12
* docker-machine → https://docs.docker.com/machine/install-machine/


Swarm - create machines
=======================
## Create three machines with docker-machine

    for node in node-1 node-2 node-3; do
        docker-machine create -d virtualbox --virtualbox-memory=1024 $node
    done


    docker-machine ls
    node-1             -        virtualbox   Running   tcp://192.168.99.100:2376             v1.12.6
    node-2             -        virtualbox   Running   tcp://192.168.99.101:2376             v1.12.6
    node-3             -        virtualbox   Running   tcp://192.168.99.102:2376             v1.12.6

Swarm - create cluster
========================

## Connect to node-1 and init the swarm master

    eval $(docker-machine env node-1)
    docker swarm init --advertise-addr $(docker-machine ip node-1) \
    --listen-addr $(docker-machine ip node-1):2377

## Join the master

    TOKEN=$(docker swarm join-token -q worker)
    for node in node-2 node-3; do
        eval $(docker-machine env $node)
        docker swarm join --token $TOKEN $(docker-machine ip node-1):2377
    done

Swarm - create networks
=========================

## create the overlay netwoks

    eval $(docker-machine env node-1)
    docker network create --driver overlay --subnet 10.10.1.0/24 logging
    docker network create --driver overlay --subnet 10.20.1.0/24 web

Swarm - create service
=======================

## Create web server

    eval $(docker-machine env node-1)
    docker service create --name webserver \
      --network logging,web -p 8080:80 nginx

    docker service create --name redis \
      --network logging,web redis

Swarm - scale services
===========================

## scale up

    docker service update --replicas 2 webserver

## scale down

    docker service update --replicas 1 webserver


Further reading
===============
[docker.com](https://www.docker.com)

[Registry](https://hub.docker.com/)

[Commandline reference](https://docs.docker.com/reference/commandline/cli/)

[Dockerfile reference](https://docs.docker.com/reference/builder/)

[Networking](https://docs.docker.com/engine/userguide/networking/)

[docker-compose](https://docs.docker.com/compose/)

[Swarm](https://docs.docker.com/engine/swarm/)
