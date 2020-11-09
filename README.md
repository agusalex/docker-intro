Docker Intro
===============

Intro a Docker

Slides: <https://agusalex.github.io/docker-intro>

Docker
===============
1. __Intro__
1. __CLI__
1. __Buildear__
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

Stop:

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
copia archivos desde y hacia el container, e.g.

    docker cp my_webserver:/etc/nginx/nginx.conf ~/

Ejercicio
==========================

Ejercicio
----------
1. Arrancar un webserver
2. Reemplazar el archivo index.html
3. Ver los logs del webserver


Build Dockerfiles
==========================
La manera estandard de crear imagenes es atravez de un `Dockerfile`

1. Crear un `Dockerfile`, e.g.

        FROM nginx
        COPY index.html /usr/share/nginx/html/

2. Buildearlo y darle un nombre

        docker build -t my-nginx .


Note:
---------------------
- El build va a tener el directorio actual como contexto
- Todas las rutas van a ser relativa al Dockerfile
- Cada comando va a creer una imagen temporal (layer)
- Cada paso se tiene en cache


Dockerfile
=====================

FROM
--------

Setear la imagen base `FROM` :

    FROM <image>:<tag>

Ejemplo:

    FROM nginx:15:04

Dockerfile
=====================

COPY
--------
`COPY` Se usa para copiar archivos desde el host hacia la imagen durante el building

    COPY <src>... <dest>

- Uso de * wildcards
- Crea el dir si este no existe

Ejemplo:

    COPY service.config /etc/service/
    COPY service.config /etc/service/myconfig.cfg
    COPY *.config /etc/service/
    COPY cfg/ /etc/service/

Dockerfile
=====================

Ejercicio
----------

Recrear el webserver esta vez usando `docker build`

Dockerfile
=====================

CMD
--------

Con `CMD` se puede especificar los comandos a ejecutar cuando arranque el container

    CMD ["executable","param1","param2"]

Ejemplo:

    CMD ["nginx", "-g", "daemon off;"]

Dockerfile
=====================

ENTRYPOINT
-----------

 `ENTRYPOINT` es como `CMD` pero mas estricto, solo permite que le pases parametros pero no puede ser reemplazado

- Toma lo que esta en `CMD` como argumento

    ENTRYPOINT ["executable", "param1", "param2"]

Example:

    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

Dockerfile
=====================

RUN
--------

`RUN` ejecuta comandos arbitrarios en el container, modificando el
filesystem y produciendo una nueva capa.

    RUN <command>

Para reducir cantidad de layers conviene agrupar comandos

Example:

    RUN apt-get update && \
        apt-get install -y ca-certificates nginx=${NGINX_VERSION} && \
        rm -rf /var/lib/apt/lists/*

Ejemplo con NGINX
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


Leer mas
===============
[docker.com](https://www.docker.com)

[Registry](https://hub.docker.com/)

[Commandline reference](https://docs.docker.com/reference/commandline/cli/)

[Dockerfile reference](https://docs.docker.com/reference/builder/)

[Networking](https://docs.docker.com/engine/userguide/networking/)

[docker-compose](https://docs.docker.com/compose/)

[Swarm](https://docs.docker.com/engine/swarm/)
