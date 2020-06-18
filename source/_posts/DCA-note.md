---
title: DCA Note
date: 2020-06-11 16:38:24
tags: docker, DCA
toc: true
---

# Docker Certified Associate Exam Preparation Guide (v1.3)

## Domain 1: Orchestration (25% of exam)

- [Complete the setup of a swarm mode cluster, with managers and worker nodes](https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/)

``` bash
$ docker swarm init
$ docker swarm join-token manager / $ docker swarm join-token worker
$ docker swarm join --token <token> <ip>:<port>
```

- [Describe and demonstrate how to extend the instructions to run individual containers into running services under swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/)

``` bash
$ docker service create --replicas 1 --name <service> <image> <cmd>
$ docker service ls
```

- [Describe the importance of quorum in a swarm cluster.](https://docs.docker.com/engine/swarm/raft/)

    - [Raft Consensus Algorithm](http://thesecretlivesofdata.com/raft/) to make sure all the manager nodes in charge of managing and scheduling tasks in the cluster, are storing the same consistent state. 
    - Having the same consistent state across the cluster means that in case of a failure, any Manager node can pick up the tasks and restore the services to a stable state.
    - Raft tolerates up to (N-1)/2 failures and requires a majority or quorum of (N/2)+1 members to agree on values proposed to the cluster.

- [Describe the difference between running a container and running a service.](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#services-tasks-and-containers)

Service -> tasks - containers

- [Interpret the output of “docker inspect” commands](https://docs.docker.com/engine/swarm/swarm-tutorial/inspect-service/)

``` bash
$ docker service|container|network|... inpsect --pretty <name>
```

- [Convert an application deployment into a stack file using a YAML compose file with "docker stack deploy"](https://docs.docker.com/engine/swarm/stack-deploy/)

stack file is a YAML defining services, networks and volumes.

- [Manipulate a running stack of services](https://docs.docker.com/engine/reference/commandline/stack_services/#related-commands)

`docker service scale`; `docker service update`; update config file, re-deploy by: `docker stack deploy -c <stackfile> <stack>`

- [Describe and demonstrate orchestration activities](https://docs.docker.com/get-started/orchestration/)

Tools to manage, scale, and maintain containerized applications are called orchestrators (e.x. Kubernetes and Docker Swarm)

- [Increase number of replicas](https://docs.docker.com/engine/reference/commandline/service_scale/)

```bash
$ docker service scale <service>=20
```
- [Add networks, publish ports](https://docs.docker.com/network/) 

``` bash
$ docker network create -d <driver> <network>
$ docker container create --network <network> -p 8080:8080 <container>
```

- [Mount volumes](https://docs.docker.com/storage/volumes/)

to attach volume to a container, either by --mount or -v:

```bash
$ docker run -d --name <container> --mount source=<volume>,target=/vol <image>
$ docker run -d --name <container> -v <volume>:/vol <image>
```

- [Describe and demonstrate how to run replicated and global services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#replicated-and-global-services)

```bash
$ docker service create --replicas 5 <service> # default replicated mode
$ docker service create --mode global <service> # global mode
```

- [Apply node labels to demonstrate placement of tasks](https://success.docker.com/article/using-contraints-and-labels-to-control-the-placement-of-containers)

- [Describe and demonstrate how to use templates with “docker service create”](https://docs.docker.com/engine/reference/commandline/service_create/#create-services-using-templates)

only `--hostname`,`--mount`,`--env` support templates, e.x.:
```bash
$ docker service create \
    --name hosttempl \
    --hostname="{{.Node.Hostname}}-{{.Node.ID}}-{{.Service.Name}}" \ # default container ID
    busybox top
```

- [Identify the steps needed to troubleshoot a service not deploying](https://success.docker.com/article/swarm-troubleshooting-methodology)

```bash
$ docker service ls
$ docker service ps <service>
$ docker service inspect <service>
$ docker inspect <task>
$ docker inspect <container>
$ docker logs <container>
```

- [Describe how a Dockerized application communicates with legacy systems](https://docs.docker.com/config/containers/container-networking/)

use a bridge, an overlay, a macvlan network, or a custom network plugin.

- [Describe how to deploy containerized workloads as Kubernetes pods and deployments](https://docs.docker.com/get-started/kube-deploy/)

```bash
$ kubectl apply -f bb.yaml
$ kubectl get deployments # docker stack ps <stack>
$ kubectl get services # docker service ls
$ kubectl delete -f bb.yaml
```

- [Describe how to provide configuration to Kubernetes pods using configMaps and secrets](https://opensource.com/article/19/6/introduction-kubernetes-secrets-and-configmaps)

## Domain 2: Image Creation, Management, and Registry (20% of exam)
- [Describe Dockerfile options (add, copy, volumes, expose, entrypoint, etc)](https://docs.docker.com/engine/reference/builder/#from)
- [Show the main parts of a Dockerfile](https://docs.docker.com/engine/reference/builder/#dockerfile-examples)
- [Give examples on how to create an efficient image via a Dockerfile](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
- [Use CLI commands such as list, delete, prune, rmi, etc to manage images](https://docs.docker.com/engine/reference/commandline/image/#usage)
- [Inspect images and report specific attributes using filter and format](https://docs.docker.com/engine/reference/commandline/inspect/#extended-description)
- [Demonstrate tagging an image](https://docs.docker.com/engine/reference/commandline/tag/)
- [Utilize a registry to store an image](https://docs.docker.com/registry/deploying/#run-a-local-registry)
- [Display layers of a Docker image](https://docs.docker.com/engine/reference/commandline/image_history/)
- [Apply a file to create a Docker image](https://docs.docker.com/engine/reference/commandline/image_load/)
- [Modify an image to a single layer](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#minimize-the-number-of-layers)
- [Describe how image layers work](https://docs.docker.com/storage/storagedriver/#images-and-layers)
- [Deploy a registry (not architect)](https://docs.docker.com/registry/deploying/)
- [Configure a registry](https://docs.docker.com/registry/configuration/)
- [Log into a registry](https://docs.docker.com/engine/reference/commandline/login/#parent-command)
- [Utilize search in a registry](https://docs.docker.com/engine/reference/commandline/search/)
- [Tag an image](https://docs.docker.com/engine/reference/commandline/tag/)
- [Push an image to a registry](https://docs.docker.com/engine/reference/commandline/push/)
- [Sign an image in a registry](https://docs.docker.com/datacenter/dtr/2.4/guides/user/manage-images/sign-images/)
- [Pull an image from a registry](https://docs.docker.com/engine/reference/commandline/pull/)
- Describe how image deletion works. [Pruning](https://docs.docker.com/config/pruning/) and [removing](https://docs.docker.com/engine/reference/commandline/rmi/)
- [Delete an image from a registry](https://docs.docker.com/datacenter/dtr/2.0/repos-and-images/delete-an-image/)

## Domain 3: Installation and Configuration (15% of exam)
- [Demonstrate the ability to upgrade the Docker engine](https://docs.docker.com/install/linux/docker-ce/ubuntu/#upgrade-docker-engine---community)
- [Complete setup of repo, select a storage driver, and complete installation of Docker
engine on multiple platforms](https://docs.docker.com/install/)
- [Configure logging drivers (splunk, journald, etc)](https://docs.docker.com/config/containers/logging/configure/)
- [Setup swarm, configure managers, add nodes, and setup backup schedule](https://docs.docker.com/engine/swarm/admin_guide/)
- [Create and manage user and teams](https://docs.docker.com/datacenter/dtr/2.4/guides/admin/manage-users/create-and-manage-teams/)
- [Interpret errors to troubleshoot installation issues without assistance](https://docs.docker.com/config/daemon/#troubleshoot-the-daemon)
- [Outline the sizing requirements prior to installation](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/install/system-requirements/#hardware-and-software-requirements)
- [Understand namespaces, cgroups, and configuration of certificates](https://docs.docker.com/engine/docker-overview/#namespaces)
- [Use certificate-based client-server authentication to ensure a Docker daemon has the
rights to access images on a registry](https://docs.docker.com/engine/security/certificates/)
- Consistently repeat steps to deploy Docker engine, UCP, and DTR on AWS and on
premises in an HA config. [Docker,](https://docs.docker.com/install/linux/docker-ce/ubuntu/) [DTR,](https://docs.docker.com/datacenter/dtr/2.3/guides/admin/install/) [UCP,](https://docs.docker.com/ee/ucp/), [Docker on AWS](https://docs.docker.com/docker-for-aws/) and possibly [on premises HA config](https://docs.docker.com/engine/swarm/admin_guide/#add-manager-nodes-for-fault-tolerance)
- [Complete configuration of backups for UCP and DTR](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/backups-and-disaster-recovery/)
- [Configure the Docker daemon to start on boot](https://docs.docker.com/install/linux/linux-postinstall/)

## Domain 4: Networking (15% of exam)
- [Create a Docker bridge network for a developer to use for their containers](https://docs.docker.com/network/network-tutorial-standalone/)
- [Troubleshoot container and engine logs to understand a connectivity issue between
containers](https://success.docker.com/article/troubleshooting-container-networking)
- [Publish a port so that an application is accessible externally](https://github.com/wsargent/docker-cheat-sheet#exposing-ports)
- [Identify which IP and port a container is externally accessible on](https://docs.docker.com/engine/reference/commandline/port/#examples)
- [Describe the different types and use cases for the built-in network drivers](https://blog.docker.com/2016/12/understanding-docker-networking-drivers-use-cases/)
- [Understand the Container Network Model and how it interfaces with the Docker engine
and network and IPAM drivers](https://success.docker.com/article/networking/)
- [Configure Docker to use external DNS](https://gist.github.com/Evalle/7b21e0357c137875a03480428a7d6bf6)
- [Use Docker to load balance HTTP/HTTPs traffic to an application (Configure L7 load
balancing with Docker EE)](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/use-a-load-balancer/#configuration-examples)
- [Understand and describe the types of traffic that flow between the Docker engine,
registry, and UCP controllers](https://success.docker.com/article/networking/)
- [Deploy a service on a Docker overlay network](https://docs.docker.com/network/overlay/)
- Describe the difference between "host" and "ingress" port publishing mode ([Host](https://docs.docker.com/engine/swarm/services/#publish-a-services-ports-directly-on-the-swarm-node), [Ingress](https://docs.docker.com/engine/swarm/ingress/))

## Domain 5: Security (15% of exam)
- [Describe the process of signing an image](https://docs.docker.com/engine/security/trust/content_trust/#push-trusted-content)
- [Demonstrate that an image passes a security scan](https://docs.docker.com/datacenter/dtr/2.5/guides/admin/configure/set-up-vulnerability-scans/)
- [Enable Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/)
- [Configure RBAC in UCP](https://docs.docker.com/datacenter/ucp/2.2/guides/access-control/)
- [Integrate UCP with LDAP/AD](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/external-auth/)
- [Demonstrate creation of UCP client bundles](https://blog.docker.com/2017/09/get-familiar-docker-enterprise-edition-client-bundles/)
- [Describe default engine security](https://docs.docker.com/engine/security/security/)
- [Describe swarm default security](https://docs.docker.com/engine/swarm/how-swarm-mode-works/pki/)
- [Describe MTLS](https://diogomonica.com/2017/01/11/hitless-tls-certificate-rotation-in-go/)
- [Identity roles](https://docs.docker.com/datacenter/ucp/2.2/guides/access-control/permission-levels/#roles)
- [Describe the difference between UCP workers and managers](https://docs.docker.com/datacenter/ucp/2.2/guides/architecture/)
- Describe process to use external certificates with UCP and DTR (**UCP** [from cli](https://success.docker.com/article/how-do-i-provide-an-externally-generated-security-certificate-during-the-ucp-command-line-installation), [from GUI](https://docs.docker.com/ee/ucp/admin/configure/use-your-own-tls-certificates/#configure-ucp-to-use-your-own-tls-certificates-and-keys), [print the public certificates](https://docs.docker.com/datacenter/ucp/3.0/reference/cli/dump-certs/)), [**DTR**](https://docs.docker.com/ee/dtr/admin/configure/use-your-own-tls-certificates/))

## Domain 6: Storage and Volumes (10% of exam)
- [State which graph driver should be used on which OS](https://docs.docker.com/storage/storagedriver/select-storage-driver/)
- [Demonstrate how to configure devicemapper](https://docs.docker.com/storage/storagedriver/device-mapper-driver/#configure-docker-with-the-devicemapper-storage-driver)
- [Compare object storage to block storage, and explain which one is preferable when
available](https://rancher.com/block-object-file-storage-containers/)
- [Summarize how an application is composed of layers and where those layers reside on
the filesystem](https://docs.docker.com/storage/storagedriver/#images-and-layers)
- [Describe how volumes are used with Docker for persistent storage](https://docs.docker.com/storage/volumes/)
- Identify the steps you would take to clean up unused images on a filesystem, also on DTR.
([image prune](https://docs.docker.com/engine/reference/commandline/image_prune/), [system prune](https://docs.docker.com/engine/reference/commandline/system_prune/) and [from DTR](https://docs.docker.com/ee/dtr/user/manage-images/delete-images/))
- [Demonstrate how storage can be used across cluster nodes](https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins)

