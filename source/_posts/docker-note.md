---
title: Docker Note
date: 2020-06-08 13:40:24
tags: docker
toc: true
---

# Install Docker

## On Debian

uninstall old versions:

``` bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

``` bash
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

Verfiy key with the fingerprint:

``` bash
sudo apt-key fingerprint 0EBFCD88
```

For x86_64 / amd64 architecture:

``` bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

``` bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Run hello world demo

``` bash
$ sudo docker run hello-world
```

Avoid `sudo` when running docker:

``` bash
$ sudo nano /etc/group
```

add username to `docker:x:999:<username>`, e.x. `docker:x:999:admin`

## On Ubuntu by Shell Script

``` bash
$ wget -qO- https://get.docker.com/ | sh
```

add user accout to local unix docker group, to avoid `sudo`:

``` bash
sudo usermod -aG docker <username> # e.x. admin
```

## Launching a Docker container

``` bash
nano Dockerfile
```

``` Dockerfile
##############################
# Dockerfile to create Ubuntu webserver
#
FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install -y apache2
RUN echo "Welcome to my web site" > /var/www/html/index.html
EXPOSE 80
##############################
```

``` bash
docker build -t "webserver" .
docker images
docker run -d -p 80:80 webserver /usr/sbin/apache2ctl -D FOREGROUND
docker ps
curl localhost
```

---

# Docker Engine Architecture

## Big Picture

![docker_engine](/photos/docker-note/docker_engine.png)

## Example of creating a container

![create_container](/photos/docker-note/create_container.png)

"Daemon" can restart without affecting on containers, which means upgrading doesn't kill containers, same for "containerd". Can restart them, leaving all containers running, when come back, they re-discover running containers and reconnect to the shim.

## Docker on Windows: Native & Hyper-V

![windows_container](/photos/docker-note/windows_container.png)

only low level difference, APIs for users are the same.
The idea is by VM isolation of Hyper-V(lightweight VM) might be better or more secure than Namespaces. Also, can run different OS in VM.

---

# Docker Images

![docker_image](/photos/docker-note/docker_image.png)

Image is **read-only** template for creating application containers. **Independent** layer loosely connected by a manifest file(config file).

![docker_image_container](/photos/docker-note/docker_image_container.png)

Every container can also write by copying read-only layer(files in it) to its writable layer, and do changes there.

## Pull a docker image

``` bash
$ docker image pull redis
```

In fact, we are pulling layers:

``` bash
Using default tag: latest
latest: Pulling from library/redis
afb6ec6fdc1c: Pull complete # layer
608641ee4c3f: Pull complete 
668ab9e1f4bc: Pull complete
78a12698914e: Pull complete
d056855f4300: Pull complete
618fdf7d0dec: Pull complete
Digest: sha256:ec277acf143340fa338f0b1a9b2f23632335d2096940d8e754474e21476eae32
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

Fat manifest is specified by OS architecture, to get image manifest:

![image_manifest](/photos/docker-note/image_manifest.png)

Referencing by hash to avoid mismatch between the image asking for and the image got.

Using overlay2 as storage driver, those layers are stored at `/var/lib/docker/overlay2`. To see layers, run:

``` bash
$ sudo ls -l /var/lib/docker/overlay2
```

to get content of layer, run:

``` bash
$ sudo ls -l /var/lib/docker/overlay2/<sha256>
```

Layer structure, e.x.:

1. Base layer (OS files & objects)
2. App codes
3. Updates ...

-> a single unified file system.

## Check images

``` bash
$ docker image ls  / $ docker images
$ docker image ls --digests # get sha256
```

to see operation history of one image, run:

``` bash
$ docker history redis
```

every non-zero size creates a new layer, the rests add something to image's json config file. 

to get configs and layers of one image, run:

``` bash
$ docker image inspect redis
```

## Delete images

``` bash
$ docker image rm redis
$ docker rmi alpine
```

## Registries

Images live in registries. When `docker image pull <some-image>`, defaultly pulling from Docker Hub.

Official images live in the top level of Docker Hub namespaces, e.x. docker.io/redis, docker.io/nginx. Can ignore registry name "docker.io/" by default, then do repo name "redis", then do tag name "latest", which is an image actually. So the full version is `docker image pull docker.io/redis:4.0.1

Unofficial ones, e.x. nigelpoulton/tu-demo

To pull all tags of images from repo, run

``` bash
$ docker image pull <some-image> -a
```

Content hashes for host, compressed hashes(distribution hashes) for wire, to verify. UIDs used for storing layers are random.

run sha256 on content of layer -> layer's hash as ID; 
run sha256 on image config file -> image's hash as ID.

---

# Containerizing

## Dockerfile

Dockerfile is list of instructions for building images with an app inside(document the app). 

Good practice: put Dockerfile in root folder of app.

Good practice: `LABEL maintainer="xxx@gmail.com"`

notes:

- CAPITALIZE instructions
- `<INSTRUCTION> <value>`, e.x. `FROM alpine`
- `FROM` always first instruction, as base image
- `RUN` execute command and create layer
- `COPY` copy code into image as new layer
- instructions like `WORKDIR` are adding metadata instead of layers
- `ENTRYPOINT` default app for image/container, metadata
- `CMD` run-time arguments override CMD instructions, append to `ENTRYPOINT`

e.x.:

``` Dockerfile
FROM alpine

LABEL maintainer="xyshell@bu.edu"

RUN apk add --update nodejs nodejs-npm

COPY . /src

WORKDIR /src

RUN npm install

EXPOSE 8080

ENTRYPOINT ["node", "./app.js"] # relative to WORKDIR
```

code: https://github.com/nigelpoulton/psweb

## Build image

``` bash
$ docker image build -t <image-name> . # current folder
```

``` bash
$ docker image build -t psweb https://github.com/nigelpoulton/psweb.git # git repo
```

`docker image ls` to check it exists.

During each step, docker spins up temporary containers, once the following layer is built, the container is removed.

## Run container

``` bash 
$ docker container run -d --name <app-name> -p 8080:8080 <image>
```

## Multi-stage Builds

`FROM ... AS ...`

`COPY --from==...`

``` Dockerfile
FROM node:latest AS storefront
WORKDIR /usr/src/atsea/app/react-app
COPY react-app .
RUN npm install
RUN npm run build

FROM maven:latest AS appserver
WORKDIR /usr/src/atsea
COPY pom.xml .
RUN mvn -B -f pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve
COPY . .
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests

FROM java:8-jdk-alpine
RUN adduser -Dh /home/gordon gordon
WORKDIR /static
COPY --from=storefront /usr/src/atsea/app/react-app/build/ . 
WORKDIR /app
COPY --from=appserver /usr/src/atsea/target/AtSea-0.0.1-SNAPSHOT.jar .
ENTRYPOINT ["java", "-jar", "/app/AtSea-0.0.1-SNAPSHOT.jar"]
CMD ["--spring.profiles.active=postgres"]
```

source: https://github.com/dockersamples/atsea-sample-shop-app/blob/master/app/Dockerfile

---

# Docker containers

Most atomic unit in docker is container.

microservices instead of monolith, glue by APIs. Containers should be as small and simple as possible. Single process per container.

modernize traditional apps:

![modernize_traditional_apps](/photos/docker-note/modernize_traditional_apps.png)

container should be ephemeral and immutable.

## Check containers

``` bash
$ docker ps / $ docker container ls # running containers
$ docker ps -a # all containers
```

## Run containers

``` bash
$ docker container run ... / docker run ...
$ docker container run -it alpine sh # iterative terminal
$ docker container run -d alpine sleep 1d # detached mode, command
```

## Stop containers

Stopping a container sends signal to main process in the container (PID1), gives 10s before force stop.

Stopping and restarting a container doesn't destory data.

``` bash
$ docker container stop <container> 
```

## Start containers

``` bash
$ docker container start <container> 
```

## Enter containers

execing into a container starts a new process

``` bash
$ docker container exec -it <container> sh
$ docker container exec <container> ls -l
$ docker container exec <container> cat <file-name>
```

exiting by `exit` kills the process, if it's the only one, container exits. However, ctrl+P+Q gets out of container without terminating its main process.

## Remove containers

``` bash
$ docker container rm <container>
$ docker container rm $(docker container ls -aq) -f # remove all containers, force
```

## Port mapping

``` bash
$ docker port <container>
```

e.x. 8080/tcp -> 0.0.0.0:8080

## Logging

Engine/daemon logs & Container/App logs

``` bash
$ docker logs <container>
```
![logging](/photos/docker-note/logging.png)
