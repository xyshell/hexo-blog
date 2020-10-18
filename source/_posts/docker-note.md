---
title: Docker Note
date: 2020-06-08 13:40:24
tags: docker
toc: true
---

# Install Docker

## On Debian

uninstall old versions:

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

```bash
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

```bash
sudo apt-key fingerprint 0EBFCD88
```

For x86_64 / amd64 architecture:

```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Run hello world demo

```bash
$ sudo docker run hello-world
```

Avoid `sudo` when running docker:

```bash
$ sudo nano /etc/group
```

add username to `docker:x:999:<username>`, e.x. `docker:x:999:admin`

Or:

```bash
sudo usermod -a -G docker <username>
```

## On Ubuntu by Shell Script

```bash
$ wget -qO- https://get.docker.com/ | sh
```

add user accout to local unix docker group, to avoid `sudo`:

```bash
sudo usermod -aG docker <username> # e.x. admin
```

## Launching a Docker container

```bash
nano Dockerfile
```

```Dockerfile
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

```bash
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

## Docker on Windows: Native and Hyper-V

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

```bash
$ docker image pull redis
```

In fact, we are pulling layers:

```bash
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

```bash
$ sudo ls -l /var/lib/docker/overlay2
```

to get content of layer, run:

```bash
$ sudo ls -l /var/lib/docker/overlay2/<sha256>
```

Layer structure, e.x.:

1. Base layer (OS files and objects)
2. App codes
3. Updates ...

-> a single unified file system.

## Check images

```bash
$ docker image ls  / $ docker images
$ docker image ls --digests # get sha256
$ docker image ls --filter dangling=true # get <none>:<none>
$ docker image ls --filter=reference="*:latest"
$ docker image ls --format "{{.Size}}"
$ docker image ls --format "{{.Repository}}: {{.Tag}}: {{.Size}}"
$ docker image ls --format "{{json .}}" # print in json format
```

to see operation history of one image, run:

```bash
$ docker history redis
```

every non-zero size creates a new layer, the rests add something to image's json config file.

to get configs and layers of one image, run:

```bash
$ docker image inspect redis
```

## Delete images

```bash
$ docker image rm redis
$ docker rmi alpine
$ docker image prune # delete all dangle images
$ docker image prune -a # delete all unused images, not used by any container
```

## Registries

Images live in registries. When `docker image pull <some-image>`, defaultly pulling from Docker Hub.

Official images live in the top level of Docker Hub namespaces, e.x. docker.io/redis, docker.io/nginx. Can ignore registry name "docker.io/" by default, then do repo name "redis", then do tag name "latest", which is an image actually. So the full version is `docker image pull docker.io/redis:4.0.1

Unofficial ones, e.x. nigelpoulton/tu-demo

To pull all tags of images from repo, run

```bash
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

```Dockerfile
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

```bash
$ docker image build -t <image-name> . # current folder
```

```bash
$ docker image build -t psweb https://github.com/nigelpoulton/psweb.git # git repo
```

`docker image ls` to check it exists.

During each step, docker spins up temporary containers, once the following layer is built, the container is removed.

## Run container

```bash
$ docker container run -d --name <app-name> -p 8080:8080 <image> # detach mode
$ docker run -dit --name alpine1 alpine ash # iterative and detach, can docker attach alpine1 later
```

## Multi-stage Builds

use multiple FROM statements in Dockerfile.

Each FROM instruction can use a different base, and each of them begins a new stage of the build.

can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image.

- `FROM ... AS ...`
- `COPY --from==...`

example 1:

```Dockerfile
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

example 2:

```Dockerfile
FROM golang:1.7.3 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html
COPY app.go    .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
CMD ["./app"]
```

source: https://docs.docker.com/develop/develop-images/multistage-build/#name-your-build-stages

When building image, don’t have to build the entire Dockerfile including every stage. Can specify a target build stage, and stop at that stage.

```bash
$ docker build --target builder -t alexellis2/href-counter:latest .
```

---

# Docker containers

Most atomic unit in docker is container.

microservices instead of monolith, glue by APIs. Containers should be as small and simple as possible. Single process per container.

modernize traditional apps:

![modernize_traditional_apps](/photos/docker-note/modernize_traditional_apps.png)

container should be ephemeral and immutable.

## Check status

```bash
$ docker ps / $ docker container ls # running containers
$ docker ps -a # all containers
$ docker port <container> # Port mapping e.x. 80/tcp -> 0.0.0.0:80
```

## Run containers

```bash
$ docker container run ... / docker run ...
$ docker container run -it alpine sh # iterative terminal
$ docker container run -d alpine sleep 1d # detached mode, command
```

## Stop containers

Stopping a container sends signal to main process in the container (PID1), gives 10s before force stop.

Stopping and restarting a container doesn't destory data.

```bash
$ docker container stop <container>
```

## Start containers

```bash
$ docker container start <container>
```

## Enter containers

execing into a container starts a new process

```bash
$ docker container exec -it <container> sh
$ docker container exec <container> ls -l
$ docker container exec <container> cat <file-name>
```

exiting by `exit` kills the process, if it's the only one, container exits. However, ctrl+P+Q gets out of container without terminating its main process (can `docker attach` to it).

## Remove containers

```bash
$ docker container rm <container>
$ docker container rm $(docker container ls -aq) -f # remove all containers, force
```

## Log

Engine/daemon logs and Container/App logs

```bash
$ docker logs <container>
```

![logging](/photos/docker-note/logging.png)

# Swarm and Services

## Swarm

swarm is a secure cluster of docker nodes, including "secure cluster" and "orchestrator"

can do native swarm work and kubernetes on swarm cluster

single-engine mode: install individual docker instances VS swarm mode: working on a cluster of docker instances.

```bash
$ docker system info #  Swarm: inactive/active
```

### Single docker node to swarm mode

```bash
$ docker swarm init
$ docker swarm init --external-ca
```

if it's the first manager of swarm, it's automatically elected as its leader(root CA).

- issue itself a client certificate.
- Build a secure cluster store(ETD) and automatically distributed to every other manager in the swarm, encrypted.
- default certificate rotation policy.
- a set of cryptographic join tokens, one for joining new managers, another for joining new workers.

on manager node, query cluster store to check all nodes:

```bash
$ docker node ls # lists all nodes in the swarm
$ docker node ls --filter role=manager/worker
```

### Join another manager and workers

```bash
$ docker swarm join
```

![swarm](/photos/docker-note/swarm.png)

Every swarm has a **single leader manager**, the rest are follower managers.

Commands could be issued to any manager, hitting a follower manager will proxy commands to the leader.

If the leader fails, another one gets selected as a new leader.

Best practice: ideal number of managers is 3, 5, 7. Make sure its odd number, to increase chance of achieving quorum.

Connect managers by fast and reliable network. e.x. in AWS, put in same region, could cross availability zones.

Workers doesn't join cluster store, which is just for managers.

Workers have a full list of IPs for all managers. If one manager dies, workders talk to another.

get join token:

```bash
$ docker swarm join-token manager
SWMTKN-1-36xjjuzeryn11xc2xtrnjjxy288aef43o2r8o8grrpela5gsq4-2pvht1x50o3s8hm5rcur5cizo 192.168.0.31:2377
$ docker swarm join-token worker
SWMTKN-1-36xjjuzeryn11xc2xtrnjjxy288aef43o2r8o8grrpela5gsq4-5rzpk82wtde7durxwiu1mmh14 192.168.0.31:2377
```

note:

- SWMTKN: identifier
- 36xjju: cluster identifier, hash of cluster certificate (same for same swarm cluster)
- 2pvht1 or 5rzpk8: determines worker or manager (could change by rotation)

switch to another node:

```bash
$ docker swarm join --token ...
```

to rotate token(change password):

```bash
$ docker swarm join-token --rotate worker
$ docker swarm join-token --rotate manager
```

The existing managers and workers stay unaffected.

to get client certificates:

```bash
$ sudo openssl x509 -in /var/lib/docker/swarm/certificates/swarm-node.crt -text
```

in Subject field:

```bash
Subject: O = pn6210vdux6ppj3uef0kqn8cv, OU = swarm-manager, CN = dkyneha22mdz384n3b3mdvjk7
```

note:

- O: organization, swarm ID
- OU: organizational unit, node's role(swarm-manager/workder)
- CN: canonical name, cryptographic node ID

### Remmove swarm node

```bash
$ docker node demote <NODE> # To demote the node
$ docker node rm <NODE> # To remove the node from the swarm
$ docker swarm leave
```

### Autolock swarm

- prevents restarted managers(not applied to workers) from automatically re-joining the swarm
- pervents accidentally restoring old copies of the swarm

```bash
$ docker swarm init --autolock # autolock new swarm
$ docker swarm update --autolock=true # autolock existing swarm
SWMKEY-1-7e7w/gsGI2iGL9dqRtY/JqOOffnP5INPRw5uME2o+hM # jot down unlock key
```

Then if manager restarts by:

```bash
$ sudo service docker restart
```

Inspecting the cluster by `docker node ls` gives raft logs saying swarm is encrypted.

re-join the swarm by:

```bash
$ docker swarm unlock
Please enter unlock key: SWMKEY-1-7e7w/gsGI2iGL9dqRtY/JqOOffnP5INPRw5uME2o+hM
```

check again by `docker node ls` to confirm.

### Update certificate expiry time

```bash
docker swarm update --cert-expiry 48h
```

check by `docker system info`:

```bash
CA Configuration:
  Expiry Duration: 2 days
```

### Update a node

Add label metadata to a node by:

```bash
$ docker node update --label-add foo worker1
$ docker node update --label-add foo --label-add bar worker1
```

then node labels used a placement constraint when creating service.

### Orchestration intro

![orchestration_intro](/photos/docker-note/orchestration_intro.png)

---

## Services

There are two types of service deployments, replicated and global:

- For a replicated service, you specify the number of identical tasks you want to run. For example, you decide to deploy an HTTP service with three replicas, each serving the same content.
- A global service is a service that runs one task on every node. There is no pre-specified number of tasks. Each time you add a node to the swarm, the orchestrator creates a task and the scheduler assigns the task to the new node. Good candidates for global services are monitoring agents, an anti-virus scanners or other types of containers that you want to run on every node in the swarm.

### Create service

```bash
$ docker service create --replicas 5 <service> # default replicated mode
$ docker service create --mode global <service> # global mode
```

### Check status

```bash
$ docker service ls # list all services
$ docker service ps <service> # list all tasks
$ docker service inspect <service> --pretty # details
$ docker service inspect <service> | jq -r '.[].CreatedAt' 
$ docker service ps <service> --format "{{json .}}" --filter "desired-state=running" | jq -r .ID
$ docker inspect <task> | jq -r '.[].Status.ContainerStatus.ContainerID'
```

### Remove services

```bash
$ docker service rm $(docker service ls -q) # remove all services
```

### Update services

rescale services by:

```bash
$ docker service scale <service>=20
```

update service's image by:

```bash
$ docker service update \
  --image nigelpoulton/tu-demo:v2 \
  --update-parallelism 2 \
  --update-delay 20s <service>
```

then update parallelism and update delay settings are now part of the service definition.

update service's network by:

```bash
$ docker service update \
  --network-add <new-network> \
  --network-rm <old-network \
  <service>
```

### Logs

```bash
$ docker service logs <service>
  --follow
  --tail 1000
  --details
```

# Container Networking

See the current network:

```bash
$ docker network ls
```

Every container goes onto bridge (nat on Windows) network by default.

![container_network](/photos/docker-note/container_network.png)

## Network types

containers talk to each other, VMs, physicals and internet. Vice versa.

### Bridge Networking

a.k.a single-host networking, docker0.

can only connect containers on the same host. Isolated layer-two network, even on the same host. Get in/out traffic by mapping port to the host.

![bridge_network](/photos/docker-note/bridge_network.png)

```bash
$ docker network inspect bridge # default bridge network
```

```json
{
  "Name": "bridge", //
  "Id": "f904473bbf1f625413c0cd2e1b7c0271253056709731cab4271ee95906ef270c", //
  "Created": "2020-06-09T21:03:39.158623688Z",
  "Scope": "local", //
  "Driver": "bridge", //
  "EnableIPv6": false,
  "IPAM": {
    "Driver": "default",
    "Options": null,
    "Config": [
      {
        "Subnet": "172.17.0.0/16" // ip range
      }
    ]
  },
  "Internal": false,
  "Attachable": false,
  "Ingress": false,
  "ConfigFrom": {
    "Network": ""
  },
  "ConfigOnly": false,
  "Containers": {}, // currently no containers
  "Options": {
    "com.docker.network.bridge.default_bridge": "true",
    "com.docker.network.bridge.enable_icc": "true",
    "com.docker.network.bridge.enable_ip_masquerade": "true",
    "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
    "com.docker.network.bridge.name": "docker0",
    "com.docker.network.driver.mtu": "1500"
  },
  "Labels": {}
}
```

create a container without specifying network by `$ docker container run --rm -d alpine sleep 1d`, then inspect again:

```json
{
  "Containers": {
    "4ea75ff740542150570357239d2f61f236f18d3840cffb3bdeae1df2745a9c2e": {
      "Name": "cool_haibt",
      "EndpointID": "f0dc9a4302cd78d262a1ec562fae8ae635d2cebb2733c60cfa40d5eb00d04564",
      "MacAddress": "02:42:ac:11:00:02",
      "IPv4Address": "172.17.0.2/16", //ip address
      "IPv6Address": ""
    }
  }
}
```

to talk to outside, need port mapping:

```bash
$ docker container run --rm -d --name web -p 8080:80 nginx # host port 8080 to container port 80
# --rm: remove the container once it exits/stops
```

show port mapping by:

```bash
$ docker port <container>
80/tcp -> 0.0.0.0:8080 # container port -> host port
```

then can visit the web by `localhost:8080`

create a bridge network:

```bash
$ docker network create -d bridge <network-name>
```

check by `docker network ls`. To run containers in it:

```bash
$ docker container run --rm -d --network <network-name> alpine sleep 1d
```

to switch container between networks:

```bash
$ docker network disconnect <network1> web # even when container is running
$ docker network connect <network2> web
```

### Overlay Networking

a.k.a multi-host networking.

Single layer-two network spanning multiple hosts

```bash
$ docker network create
$ docker network create -o encrypted # encrypt data plane
```

built-in overlay is container to container only (not applied to VM, physicals)

![overlay_network](/photos/docker-note/overlay_network.png)

To create a overlay network:

```bash
$ docker network create -d overlay <network-name>
```

check by `docker network ls`, note its scope is "swarm", which means availabel on every node in the swarm.

create a service to use this overlay network:

```bash
$ docker service create -d --name <service> --replicas 2 --network overnet alpine sleep 1d # default replicated mode
```

check by `docker service ls`, check which nodes are running the service by `docker service ps <service>`, more details run `docker service inspect <service>`

switch to one node which runs this service and run `docker network inspect <network-name>`

```json
{
  "Containers": {
    "7df4738446ac44a577d026a11ad73401c6cbdaaafcddbe75028954e7191fe1a1": {
      "Name": "pinger.1.neku7xixs6g8oe2r8otlnqnep",
      "EndpointID": "9a2dbdda2c15b991eeba7c1ab6fabd93e42b5ed4495b2f47d038dc871143a409",
      "MacAddress": "02:42:0a:00:01:04",
      "IPv4Address": "10.0.1.4/24", // jot down ip address
      "IPv6Address": ""
    },
    "lb-overnet": {
      "Name": "overnet-endpoint",
      "EndpointID": "45c5d95a4d21eadcd2f99d0ff982ec4bc6d0c6a7f136136a93aeeb8c7d959898",
      "MacAddress": "02:42:0a:00:01:06",
      "IPv4Address": "10.0.1.6/24",
      "IPv6Address": ""
    }
  }
}
```

switch to the other node, exec into the container by `docker container exec -it <container> sh`, `ping 10.0.1.4`, check success.

### MACVLAN

Containers also need to talk to VMs or physicals on existing VLANs.

Gives every container its own IP address and MAC address on the existing network (directly on the wire, no bridges, no port mapping)

requires promiscuous mode on the host. (cloud providers generally don't allow. look for IPVLAN instead, which doesn't require promiscuous mode)

## Network services

- Service discovery: locate services in a swarm

- Load Balancing: access a service from any node in swarm (even nodes not hosting the service)

### Service discovery

Every service gets a name, registered with swarm DNS, uses swarm DNS

```bash
$ docker service create -d --name ping --network <overlay> --replicas 3 alpine sleep 1d
$ docker service create -d --name pong --network <overlay> --replicas 3 alpine sleep 1d
```

check `docker service ls` to confirm (also check `docker container ls`), can locate other service in same overlay network by name, e.x.:

```bash
$ docker container exec -it <container> sh
$ ping pong # sucesss
$ ping -c 2 pong # -c 2: only two ping attempts.
```

### Load Balancing

```bash
$ docker service create -d --name web --network overnet --replicas 1 -p 8080:80 nginx
$ docker service inspect web --pretty
Ports:
 PublishedPort = 8080 #
  Protocol = tcp
  TargetPort = 80 #
  PublishMode = ingress # default mode
```

ingress mode: publish a port on every node in the swarm — even nodes not running service replicas. then can access by any node in the network by port 8080.

The alternative mode is host mode which only publishes the service on swarm nodes running replicas. by adding mode=host to the --publish output, using --mode global instead of --replicas=5, since only one service task can bind a given port on a given node.

# Volumes

running a new container automatically gets its own non-persistent, ephemeral graph driver storage (copy-on-write union mount, /var/lib/docker). However, volume is to store persistent data, entirely decoupled from containers, seamlessly plugs into containers.

a directory on the docker(also remote hosts or cloud providers by volume drivers), mounted into container at a specific mount point.

can exist not only on local storage of docker host, but also on high-end external systems like SAN and NAS. Pluggable by docker store drive.

## Create Volumes

```bash
$ docker volume create <volume>
```

## Check status

```bash
$ docker volume ls
$ docker volume inspect <volume>
```

```json
[
  {
    "CreatedAt": "2020-06-10T16:29:30Z",
    "Driver": "local", // default
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/myvol/_data", // inspect by ls -l /var/lib/docker/volumes/
    "Name": "myvol",
    "Options": {},
    "Scope": "local" //
  }
]
```

## Delete volume

to delete a specific volume:

```bash
$ docker volume rm <volume>
```

rm an in-use volume causes error message.

to delete all unused volumes:

```bash
$ docker volume prune # delete unused volume
```

## Attach volume

to attach volume to a container, either by --mount or -v:

```bash
$ docker run -d --name <container> --mount source=<volume>,target=/vol <image>
$ docker run -d --name <container> -v <volume>:/vol <image>
```

note:

- `source=<volume>`: if volume doesn't exist for now, will be created.
- `target=/vol`: where in the container to mount it, check by exec into container and `ls -l /vol/`
- if the container has files or directories in the directory to be mounted, the directory’s contents are copied into the volume.

check by `docker inspect <container>`:

```json
"Mounts": [
  {
    "Type": "volume",
    "Name": "myvol",
    "Source": "/var/lib/docker/volumes/myvol/_data",
    "Destination": "/vol",
    "Driver": "local",
    "Mode": "",
    "RW": true,
    "Propagation": ""
  }
],
```

Then, container can write data to `/vol` (e.x. `echo "some data" > /vol/newfile`), accessible in `/var/lib/docker/volumes/` as well, even if the container is removed.

--mount works with service as well.

Also useful in Dockerfile's volume instruction.

## Volume for service

```bash
$ docker service create -d \
  --replicas=4 \
  --name <service> \
  --mount source=myvol,target=/app \
  <image>
```

note:

- docker service create command does not support the -v or --volume

## Read only volume

```bash
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```

verfify by `docker inspect nginxtest`:

```json
"Mounts": [
  {
    "Type": "volume",
    "Name": "nginx-vol",
    "Source": "/var/lib/docker/volumes/nginx-vol/_data",
    "Destination": "/usr/share/nginx/html",
    "Driver": "local",
    "Mode": "",
    "RW": false, //
    "Propagation": ""
  }
],
```

# Secrets

string <= 500k, swarm mode for services only(not containers)

![secret](/photos/docker-note/secret.png)

note:
/run/secrets/: stay in memory

```bash
$ docker secret create <secret> <file>
```

check by `docker secret ls`. inspect by `docker secret inspect <secret>`

create a service, using secret:

```bash
$ docker service create -d --name <service> --secret <secret> --replicas 2 ...
```

inpect by `docker service inspect <service>` and look at secrets section. exec into containers by `docker container exec -it <container> sh`, find secret by `ls -l /run/secrets`, accessible

can't delete an in-use secret by `docker secret rm <secret>`, need to delete service first.

# Docker Compose and Stack

## Docker compose

### Install on linux

1. Download the current stable release of Docker Compose:

```bash
$ curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2. Make it executable:

```bash
$ chmod +x /usr/local/bin/docker-compose
```

### Compose files

Compose uses YAML files to define multi-service applications.

The default name for the Compose YAML file is docker-compose.yml

Flask app example:

```yml
version: "3.5" # mandatory
services: # application services
  web-fe: # create a web front-end container called web-fe
    build: . # build a new image by Dockerfile in '.' to create container for web-fe
    command: python app.py # Override CMD in Dockerfile, which has Python and app.py
    ports:
      - target: 5000 # container port
        published: 5000 # host port
    networks:
      - counter-net # already exist or to be defined
    volumes:
      - type: volume
      source: counter-vol # already exist or to be defined
      target: /code # mount to container
  redis: # create an in-memory database container called redis
    image: "redis:alpine" # pulled from Docker Hub.
    networks:
      counter-net:

  networks: # create new networks, bridge by default
    counter-net:

  volumes: # create new volumes
    counter-vol:
```

note:

- 4 top-level keys: version, services, networks, volumes, (secrets, configs...)
- use the driver property to specify different network types:

```yml
networks:
  over-net:
  driver: overlay
  attachable: true # for standalone containers(instead of Docker Services)
```

source: https://github.com/nigelpoulton/counter-app

### Run compose app

```bash
$ docker-compose up # docker-compose.yml in current folder
$ docker-compose up & # ctrl+c doesn't kill container
$ docker-compose up -d # run in daemon
$ docker-compose -f prod-equus-bass.yml up # -f flag
```

check the current state of the app by `docker-compose ps`.

list the processes running inside of each
service (container) by `docker-compose top`.

### Stop, Restart and Delete App

stop without deleting:

```bash
$ docker-compose stop
```

could restart by:

```bash
$ docker-compose restart
```

If changed app after stopping, these changes won't apply in restarted app. Need to re-deploy.

delete a stopped Compose app and networks:

```bash
$ docker-compose rm
```

stop and delete containers and networks by:

```bash
$ docker-compose down
```

## Stack

swarm only

stacks manage a bunch of services as a single app, highest layer of docker application hierarchy.

![stack](/photos/docker-note/stack.png)

can run on Docker CLI, Docker UCP, Docker Cloud.

docker-stack.yml: YAML config file including **version**, **services**, **network**, **volumes**, documenting the app. Can do version control.

![stack_file](/photos/docker-note/stack_file.png)

```yml
version: "3" # >=3
services:
  redis: # service 1st
    image: redis:alpine #
    networks:
      - frontend
    deploy: # new in version 3
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager] # only run on manager nodes
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    depends_on:
      - db
      - redis
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

source: https://github.com/dockersamples/example-voting-app/docker-stack.yml

Check [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

Placement constraints:

- Node ID: node.id == o2p4kw2uuw2a
- Node name: node.hostname == wrk-12
- Role: node.role != manager
- Engine labels: engine.labels.operatingsystem==ubuntu 16.04
- Custom node labels: node.labels.zone == prod1 **zone: prod1**

`service.<service>.deploy.update_config`:
```yml
update_config:
  parallelism: 2 # update two replicas at-atime
  failure_action: rollback # [pause, continue, rollback]
```

`services.<service>.deploy.restart-policy`:
```yml
restart_policy:
  condition: on-failure # non-zero exit code
  delay: 5s # between each of the restart attempts
  max_attempts: 3 
  window: 120s # wait up to 120 seconds to decide if the restart worked
```

`services.<service>.stop_grace_period`:
```yml
stop_grace_period: 1m30s # for PID 1 to handle SIGTERM, default 10s, then SIGKILL.
```

### Deploy the stack

```bash
$ docker stack deploy -c <stackfile> <stack> # -c: --compose-file
```

### Check status

```bash
$ docker stack ls
$ docker stack ps <stack>
$ docker stack services <stack>
```

### Update stack

update config file, and re-deploy by:

```bash
$ docker stack deploy -c <stackfile> <stack>
```

will update every service in the stack.

# Enterprise Edition(EE)

- a hardened Docker Engine
- Ops UI
- Secure on-premises registry

![EECE](/photos/docker-note/EECE.png)

## Universal Control Plane(UCP)

based on EE, the operations GUI from Docker Inc, to manage swarm and k8s apps.

![UCP](/photos/docker-note/UCP.png)

## Docker Trusted Registry(DTR)

based on EE and UCP, a registry to store images, a containerized app.

![DTR](/photos/docker-note/DTR.png)

## Role-based Access Control(RBAC)

- subject: user, team
- role: permissions
- collection: resources(docker node)

![RBAC](/photos/docker-note/RBAC.png)

## Image scanning

after update the image in local, need to push into registry.

Tag the updated image:

```bash
$ docker image tag <image> <dtr-dns>/<repo>/<image>:latest
```

check by `docker image ls` to see a new tagged image.

login:

```bash
$  docker login <dtr-dns> # username, passwork, user needs permission to write in the repo
```

ensure image scanning sets to "scan on push" in UCP's DTR's repo setting. Then push:

```bash
$ docker image push <tagged-image>
```

Check in DTR's repo's images' vulnerabilities field.

## HTTP Routing Mesh(HRM)

For Docker CE's Routing Mesh(Swarm-mode Routing Mesh), Transport layer(L4).

![routing_mesh](/photos/docker-note/routing_mesh.png)

For Docker EE's HRM, Application layer(L7). Route based on host header.

![HRM](/photos/docker-note/HRM.png)
