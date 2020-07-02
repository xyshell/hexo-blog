---
title: DCA Note
date: 2020-06-11 16:38:24
tags: docker, DCA
toc: true
---

# Docker Certified Associate Exam Preparation Guide (v1.3)

## Domain 1: Orchestration (25% of exam)

### [Complete the setup of a swarm mode cluster, with managers and worker nodes](https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/)

```bash
$ docker swarm init
$ docker swarm join-token manager / $ docker swarm join-token worker
$ docker swarm join --token <token> <ip>:<port>
```

### [Describe and demonstrate how to extend the instructions to run individual containers into running services under swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/)

```bash
$ docker service create --replicas 1 --name <service> <image> <cmd>
$ docker service ls
```

### [Describe the importance of quorum in a swarm cluster.](https://docs.docker.com/engine/swarm/raft/)

- [Raft Consensus Algorithm](http://thesecretlivesofdata.com/raft/) to make sure all the manager nodes in charge of managing and scheduling tasks in the cluster, are storing the same consistent state.
- Having the same consistent state across the cluster means that in case of a failure, any Manager node can pick up the tasks and restore the services to a stable state.
- Raft tolerates up to (N-1)/2 failures and requires a majority or quorum of (N/2)+1 members to agree on values proposed to the cluster.

### [Describe the difference between running a container and running a service.](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#services-tasks-and-containers)

Service -> tasks - containers

### [Interpret the output of “docker inspect” commands](https://docs.docker.com/engine/swarm/swarm-tutorial/inspect-service/)

```bash
$ docker service|container|network|... inpsect --pretty <name>
```

### [Convert an application deployment into a stack file using a YAML compose file with "docker stack deploy"](https://docs.docker.com/engine/swarm/stack-deploy/)

stack file is a YAML defining services, networks and volumes.

### [Manipulate a running stack of services](https://docs.docker.com/engine/reference/commandline/stack_services/#related-commands)

`docker service scale`; `docker service update`; update config file, re-deploy by: `docker stack deploy -c <stackfile> <stack>`

### [Describe and demonstrate orchestration activities](https://docs.docker.com/get-started/orchestration/)

Tools to manage, scale, and maintain containerized applications are called orchestrators (e.x. Kubernetes and Docker Swarm)

### [Increase number of replicas](https://docs.docker.com/engine/reference/commandline/service_scale/)

```bash
$ docker service scale <service>=20
```

### [Add networks, publish ports](https://docs.docker.com/network/)

```bash
$ docker network create -d <driver> <network>
$ docker container create --network <network> -p 8080:8080 <container>
```

### [Mount volumes](https://docs.docker.com/storage/volumes/)

to attach volume to a container, either by --mount or -v:

```bash
$ docker run -d --name <container> --mount source=<volume>,target=/vol <image>
$ docker run -d --name <container> -v <volume>:/vol <image>
```

### [Describe and demonstrate how to run replicated and global services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#replicated-and-global-services)

```bash
$ docker service create --replicas 5 <service> # default replicated mode
$ docker service create --mode global <service> # global mode
```

### [Apply node labels to demonstrate placement of tasks](https://success.docker.com/article/using-contraints-and-labels-to-control-the-placement-of-containers)

### [Describe and demonstrate how to use templates with “docker service create”](https://docs.docker.com/engine/reference/commandline/service_create/#create-services-using-templates)

only `--hostname`,`--mount`,`--env` support templates, e.x.:

```bash
$ docker service create \
    --name hosttempl \
    --hostname="{{.Node.Hostname}}-{{.Node.ID}}-{{.Service.Name}}" \ # default container ID
    busybox top
```

### [Identify the steps needed to troubleshoot a service not deploying](https://success.docker.com/article/swarm-troubleshooting-methodology)

```bash
$ docker service ls
$ docker service ps <service>
$ docker service inspect <service>
$ docker inspect <task>
$ docker inspect <container>
$ docker logs <container>
```

### [Describe how a Dockerized application communicates with legacy systems](https://docs.docker.com/config/containers/container-networking/)

use a bridge, an overlay, a macvlan network, or a custom network plugin.

### [Describe how to deploy containerized workloads as Kubernetes pods and deployments](https://docs.docker.com/get-started/kube-deploy/)

```bash
$ kubectl apply -f bb.yaml
$ kubectl get pods
$ kubectl get deploy
$ kubectl edit deploy/mysite # edit existing objects, automatic update
$ kubectl scale --replicas=2 deploy/mysite # rescale
$ kubectl delete -f bb.yaml / kubectl delete deploy mysite
```

### [Describe how to provide configuration to Kubernetes pods using configMaps and secrets](https://opensource.com/article/19/6/introduction-kubernetes-secrets-and-configmaps)

1. Secret:

create the YAML file for the Secret, save as mysql-secret.yaml:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-root-password
type: Opaque
data:
  password: S3ViZXJuZXRlc1JvY2tzIQ==
```

create the Secret in Kubernetes:

```bash
$ kubectl apply -f mysql-secret.yaml
```

view the newly created Secret:

```bash
$ kubectl describe secret mariadb-root-password
```

view and edit the Secret:

```bash
$ kubectl edit secret <secretname>
```

could also create the secret by:

```bash
$ kubectl create secret generic mariadb-user-creds \
      --from-literal=MYSQL_USER=kubeuser\
      --from-literal=MYSQL_PASSWORD=kube-still-rocks
```

validate secrets were created and stored correctly:

```bash
# Get the username
$ kubectl get secret mariadb-user-creds -o jsonpath='{.data.MYSQL_USER}' | base64 --decode -
kubeuser

# Get the password
$ kubectl get secret mariadb-user-creds -o jsonpath='{.data.MYSQL_PASSWORD}' | base64 --decode -
kube-still-rocks
```

2. ConfigMap

create a file named max_allowed_packet.cnf:

```cnf
[mysqld]
max_allowed_packet = 64M
```

create configmap by:

```bash
$ kubectl create configmap mariadb-config --from-file=max_allowed_packet.cnf # could add multiple --from-file=<filename>
$ kubectl create configmap mariadb-config --from-file=max-packet=max_allowed_packet.cnf # set max-packet as key rather than the file name
configmap/mariadb-config created
```

Firstvalidate that the ConfigMap was created:

```bash
$ kubectl get configmap mariadb-config
```

viewed with the kubectl describe command:

```bash
$ kubectl describe cm mariadb-config
```

edit configmap mariadb-config's value:(in development)

```bash
$ kubectl edit configmap mariadb-config
```

3. Using Secrets and ConfigMaps

Create a file named mariadb-deployment.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mariadb
  name: mariadb-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
        - name: mariadb
          image: docker.io/mariadb:10.4
        #   env:
        #     - name: MYSQL_ROOT_PASSWORD
        #     valueFrom:
        #         secretKeyRef:
        #             name: mariadb-root-password
        #             key: password
        #   envFrom:
        #   - secretRef:
        #     name: mariadb-user-creds
          ports:
            - containerPort: 3306
              protocol: TCP
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mariadb-volume-1
            # - mountPath: /etc/mysql/conf.d
            #   name: mariadb-config-volume
      volumes:
        - emptyDir: {}
          name: mariadb-volume-1
        # - configMap:
        #   name: mariadb-config
        #   items:
        #     - key: max_allowed_packet.cnf
        #       path: max_allowed_packet.cnf
        #   name: mariadb-config-volume
```

add the Secrets to the Deployment as environment variables, same for ConfigMaps by using configMapRef instead of secretKeyRef:
```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mariadb-root-password
        key: password
```

or using envFrom:
```yaml
envFrom:
- secretRef:
    name: mariadb-user-creds
```

create a new MariaDB instance from the YAML file:
```yaml
$ kubectl create -f mariadb-deployment.yaml
deployment.apps/mariadb-deployment created
```

view the running MariaDB pod:
```bash
$ kubectl get pods
```

kubectl exec to the Pod, validate Secrets and ConfigMaps are in use:

```bash
$ kubectl exec -it mariadb-deployment-5465c6655c-7jfqm env |grep MYSQL
MYSQL_PASSWORD=kube-still-rocks
MYSQL_USER=kubeuser
MYSQL_ROOT_PASSWORD=KubernetesRocks!
```

check max_allowed_packet.cnf file was created in /etc/mysql/conf.d:
```bash
$ kubectl exec -it mariadb-deployment-5465c6655c-7jfqm ls /etc/mysql/conf.d
max_allowed_packet.cnf

$ kubectl exec -it mariadb-deployment-5465c6655c-7jfqm cat /etc/mysql/conf.d/max_allowed_packet.cnf
[mysqld]
max_allowed_packet = 32M
```

## Domain 2: Image Creation, Management, and Registry (20% of exam)

### [Describe the use of Dockerfile](https://docs.docker.com/engine/reference/builder/)
### [Describe options, such as add, copy, volumes, expose, entry point](https://docs.docker.com/engine/reference/builder/#from)
### [Identify and display the main parts of a Dockerfile](https://docs.docker.com/engine/reference/builder/#dockerfile-examples)
### [Describe and demonstrate how to create an efficient image via a Dockerfile](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
### [Describe and demonstrate how to use CLI commands to manage images, such as list, delete, prune, rmi](https://docs.docker.com/engine/reference/commandline/image/#usage)
### [Describe and demonstrate how to inspect images and report specific attributes using filter and format](https://docs.docker.com/engine/reference/commandline/images/#filtering)

filter:
```bash
$ docker images --filter "dangling=true"
$ docker images --filter "label=com.example.version"
$ docker images --filter "label=com.example.version=1.0"
$ docker images --filter "before=image1"
$ docker images --filter "since=image3"
$ docker images --filter=reference='busy*:*libc'
$ docker images --filter=reference='busy*:uclibc' --filter=reference='busy*:glibc'
```

format:
```bash
$ docker images --format "{{.ID}}: {{.Repository}}"
$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

### [Describe and demonstrate how to tag an image.](https://docs.docker.com/engine/reference/commandline/tag/)

```bash
$ docker tag <image-name/image-id> fedora/httpd:version1.0 
```

### [Describe and demonstrate how to apply a file to create a Docker image](https://docs.docker.com/engine/reference/commandline/image_load/)
### [Describe and demonstrate how to display layers of a Docker image](https://docs.docker.com/engine/reference/commandline/image_inspect/)
### [Describe and demonstrate how to modify an image to a single layer](https://stackoverflow.com/questions/39695031/how-make-docker-layer-to-single-layer)

### [Describe and demonstrate registry functions](https://docs.docker.com/registry/)
### [Deploy a registry](https://docs.docker.com/registry/deploying/)

deploy by:
```bash
$ docker run -d \ 
  -e REGISTRY_HTTP_ADDR=0.0.0.0:5001 \ 
  -p 5001:5001 \ 
  --restart=always \ 
  --name registry \ 
  -v /mnt/registry:/var/lib/registry \ 
  registry:2
```

stop and remove by:
```bash
$ docker container stop registry && docker container rm -v registry
```

### [Log into a registry](https://docs.docker.com/engine/reference/commandline/login/)

```bash
$ docker login localhost:8080
```

```bash
$ cat ~/my_password.txt | docker login --username foo --password-stdin
```

### [Utilize search in a registry](https://docs.docker.com/engine/reference/commandline/search/)

```bash
$ docker search busybox
$ docker search --filter is-official=true --filter stars=3 busybox
$ docker search --format "{{.Name}}: {{.StarCount}}" nginx
$ docker search --format "table {{.Name}}\t{{.IsAutomated}}\t{{.IsOfficial}}" nginx
```

### [Push an image to a registry](https://docs.docker.com/engine/reference/commandline/push/)

commit a container to image:
```bash
$ docker commit c16378f943fe rhel-httpd
```

tag and push:
```bash
$ docker tag rhel-httpd registry-host:5000/myadmin/rhel-httpd
$ docker push registry-host:5000/myadmin/rhel-httpd
```

### [Sign an image in a registry](https://docs.docker.com/engine/reference/commandline/trust_sign/)

```bash
$ docker trust sign example/trust-demo:v2
$ docker trust inspect --pretty example/trust-demo
```

### [Pull](ttps://docs.docker.com/engine/reference/commandline/pull/) and [delete](https://docs.docker.com/registry/spec/api/#deleting-an-image) images from a registry

An image may be deleted from the registry via its "name" and "reference":
```http
DELETE /v2/<name>/manifests/<reference>
```

## Domain 3: Installation and Configuration (15% of exam)

### [Describe sizing requirements for installation](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/install/system-requirements/#hardware-and-software-requirements)

the following minimum requirements for Docker UCP 2.2.4 on Linux:
• UCP Manager nodes running DTR: 8GB of RAM with 3GB of disk space
• UCP Worker nodes: 4GB of RAM with 3GB of free disk space
Recommended requirements are:
• UCP Manager nodes running DTR: 8GB RAM, 4 vCPUs, and 100GB disk space
• UCP Worker nodes: 4GB RAM 25-100GB of free disk space

### [Describe and demonstrate the setup of repo, selection of a storage driver, and installation of the Docker engine on multiple platforms](https://docs.docker.com/install/)

### [Describe and demonstrate configuration of logging drivers (splunk, journald, etc.)](https://docs.docker.com/config/containers/logging/configure/)

### [Describe and demonstrate how to set up swarm, configure managers, add nodes, and setup the backup schedule](https://docs.docker.com/engine/swarm/admin_guide/)

A Swarm backup is a copy of all the files in directory `/var/lib/docker/swarm`:

1. Stop Docker on the Swarm manager node you are performing the backup from(not a good idea to perform the backup on the leader manager). This will stop all UCP containers on the node. If UCP is configured for HA, the other managers will make sure the control plane remains available.

```bash
$ service docker stop
```

2. Backup the Swarm config, e.x.:

```bash
$ tar -czvf swarm.bkp /var/lib/docker/swarm/
tar: Removing leading `/' from member names
/var/lib/docker/swarm/
/var/lib/docker/swarm/docker-state.json
/var/lib/docker/swarm/state.json
<Snip>
```

3. Verify that the backup file exists. rotate, and store the backup file off-site according to your corporate backup policies.

```bash
$ ls -l
-rw-r--r-- 1 root root 450727 Jan 29 14:06 swarm.bkp
```

4. Restart Docker.

```bash
$ service docker restart
```

recover Swarm from a backup:

1. stop docker:
```bash
$ service docker stop
```

2. Delete any existing Swarm configuration:
```bash
$ rm -r /var/lib/docker/swarm
```

3. Restore the Swarm configuration from backup:
```bash
$ tar -zxvf swarm.bkp -C /
```

4. Initialize a new Swarm cluster:
```bash
$ docker swarm init --force-new-cluster
Swarm initialized: current node (jhsg...3l9h) is now a manager.
```

5. check by:
```bash
$ docker network ls
$ docker service ls
```

6. Add new manager and worker nodes to the Swarm, and take a fresh backup.

### [Describe and demonstrate how to create and manage user and teams](https://success.docker.com/article/rbac-example-overview)

RBAC via grant:
- Subject
- Role
- Collection

### [Describe and demonstrate how to configure the Docker daemon to start on boot](https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot)

```bash
$ sudo systemctl enable docker
```

To disable this behavior, use disable instead:
```bash
$ sudo systemctl disable docker
```

### [Describe and demonstrate how to use certificate-based client-server authentication to ensure a Docker daemon has the rights to access images on a registry](https://docs.docker.com/engine/security/certificates/)

### [Describe the use of namespaces, cgroups, and certificate configuration](https://docs.docker.com/get-started/overview/#the-underlying-technology)

### [Describe and interpret errors to troubleshoot installation issues without assistance](https://docs.docker.com/config/daemon/#troubleshoot-the-daemon)

### Describe and demonstrate the steps to deploy the Docker engine, UCP, and DTR on AWS and on-premises in an HA configuration. [Docker,](https://docs.docker.com/install/linux/docker-ce/ubuntu/) [DTR,](https://docs.docker.com/datacenter/dtr/2.3/guides/admin/install/) [UCP,](https://docs.docker.com/ee/ucp/), [Docker on AWS](https://docs.docker.com/docker-for-aws/) and possibly [on premises HA config](https://docs.docker.com/engine/swarm/admin_guide/#add-manager-nodes-for-fault-tolerance)

1. DTR:

If possible, you should run your DTR instances on dedicated nodes. You definitely
shouldn’t run user workloads on your production DTR nodes.

As with UCP, you should run an odd number of DTR instances. 3 or 5 is best for fault
tolerance. A recommended configuration for a production environment might be:
- 3 dedicated UCP managers
- 3 dedicated DTR instances
- However many worker nodes your application requirements demand

Install DTR:

1. Log on to the UCP web UI and click Admin > Admin Settings > Docker
Trusted Registry.
2. Fill out the DTR configuration form.
- DTR EXTERNAL URL: Set this to the URL of your external load balancer.
- UCP NODE: Select the name of the node you wish to install DTR on.
- Disable TLS Verification For UCP: Check this box if you’re using
self-signed certificates.
3. Copy the long command at the bottom of the form.
4. Paste the command into any UCP manager node.
The command includes the --ucp-node flag telling UCP which node to
perform the install on.
The following is an example DTR install command that matches the configuration
in Figure 16.10. It assumes that you already have a load balancer
configured at dtr.mydns.com
```bash
$ docker run -it --rm docker/dtr install \
--dtr-external-url dtr.mydns.com \
--ucp-node dtr1 \
--ucp-url https://34.252.195.122 \
--ucp-username admin --ucp-insecure-tls
```
5. Once the installation is complete, point your web browser to your load
balancer. You will be automatically logged in to DTR.

Configure DTR for high availability:

1. Log on to the DTR console and navigate to Settings.
2. Select the Storage tab and configure the shared storage backend.

DTR is now configured with a shared storage backend and ready to have additional
replicas.
1. Run the following command from a manager node in the UCP cluster.
```bash
$ docker run -it --rm \
docker/dtr:2.4.1 join \
--ucp-node dtr2 \
--existing-replica-id 47f20fb864cf \
--ucp-insecure-tls
```
2. Enter the UCP URL and port, as well as admin credentials when prompted.

### Describe and demonstrate how to configure backups for [UCP and DTR](https://success.docker.com/article/backup-restore-best-practices)

You can run the backup from any UCP manager node in the cluster, and you only
need to run the operation on one node (UCP replicates its configuration to all
manager nodes, so backing up from multiple nodes is not required).

Backing up UCP will stop all UCP containers on the manager that you’re executing
the operation on. With this in mind, you should be running a highly available UCP
cluster, and you should run the operation at a quiet time for the business.

backup UCP:
```bash
$ docker container run --log-driver none --rm -i --name ucp \
-v /var/run/docker.sock:/var/run/docker.sock \
docker/ucp:2.2.5 backup --interactive \
--passphrase "Password123" > ucp.bkp
```

recover UCP:
1. Remove any existing, and potentially corrupted, UCP installations:
```bash
$ docker container run --rm -it --name ucp \
-v /var/run/docker.sock:/var/run/docker.sock \
docker/ucp:2.2.5 uninstall-ucp --interactive
```

2. Restore UCP from the backup:
```bash
$ docker container run --rm -i --name ucp \
-v /var/run/docker.sock:/var/run/docker.sock \
docker/ucp:2.2.5 restore --passphrase "Password123" < ucp.bkp
```

3. Log on to the UCP web UI and ensure that the user created earlier is still present
(or any other UCP objects that previously existed in your environment).

Backup DTR:
As with UCP, DTR has a native backup command that is part of the Docker image
that was used to install the DTR. This native backup command will backup the DTR
configuration that is stored in a set of named volumes, and includes:
- DTR configuration
- Repository metadata
- Notary data
- Certificates

Images are not backed up as part of a native DTR backup. It is expected that
images are stored in a highly available storage backend that has its own independent
backup schedule using non-Docker tools.

Run the following command from a UCP manager node to perform a DTR backup:
```bash
$ read -sp 'ucp password: ' UCP_PASSWORD; \
docker run --log-driver none -i --rm \
--env UCP_PASSWORD=$UCP_PASSWORD \
docker/dtr:2.4.1 backup \
--ucp-insecure-tls \
--ucp-username admin \
> ucp.bkp
```

Recover DTR from backups:

Restoring DTR from backups should be a last resort, and only attempted when the
majority of replicas are down and the cluster cannot be recovered any other way.
If you have lost a single replica and the majority are still up, you should add a new
replica using the `dtr join` command.

restore from backup, the workflow is like this:
1. Stop and delete DTR on the node (might already be stopped)
```bash
$ docker run -it --rm \
docker/dtr:2.4.1 destroy \
--ucp-insecure-tls
```
2. Restore images to the shared storage backend (might not be required)
3. Restore DTR
```bash
$ read -sp 'ucp password: ' UCP_PASSWORD; \
docker run -i --rm \
--env UCP_PASSWORD=$UCP_PASSWORD \
docker/dtr:2.4.1 restore \
--ucp-url <ENTER_YOUR_ucp-url> \
--ucp-node <ENTER_DTR_NODE_hostname> \
--ucp-insecure-tls \
--ucp-username admin \
< ucp.bkp
```

## Domain 4: Networking (15% of exam)

### [Create a Docker bridge network for a developer to use for their containers](https://docs.docker.com/network/network-tutorial-standalone/)
### [Troubleshoot container and engine logs to understand a connectivity issue between
  containers](https://success.docker.com/article/troubleshooting-container-networking)
### [Publish a port so that an application is accessible externally](https://github.com/wsargent/docker-cheat-sheet#exposing-ports)
### [Identify which IP and port a container is externally accessible on](https://docs.docker.com/engine/reference/commandline/port/#examples)
### [Describe the different types and use cases for the built-in network drivers](https://blog.docker.com/2016/12/understanding-docker-networking-drivers-use-cases/)
### [Understand the Container Network Model and how it interfaces with the Docker engine
  and network and IPAM drivers](https://success.docker.com/article/networking/)
### [Configure Docker to use external DNS](https://gist.github.com/Evalle/7b21e0357c137875a03480428a7d6bf6)
### [Use Docker to load balance HTTP/HTTPs traffic to an application (Configure L7 load
  balancing with Docker EE)](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/use-a-load-balancer/#configuration-examples)
### [Understand and describe the types of traffic that flow between the Docker engine,
  registry, and UCP controllers](https://success.docker.com/article/networking/)
### [Deploy a service on a Docker overlay network](https://docs.docker.com/network/overlay/)
- Describe the difference between "host" and "ingress" port publishing mode ([Host](https://docs.docker.com/engine/swarm/services/#publish-a-services-ports-directly-on-the-swarm-node), [Ingress](https://docs.docker.com/engine/swarm/ingress/))

## Domain 5: Security (15% of exam)

### [Describe the process of signing an image](https://docs.docker.com/engine/security/trust/content_trust/#push-trusted-content)
### [Demonstrate that an image passes a security scan](https://docs.docker.com/datacenter/dtr/2.5/guides/admin/configure/set-up-vulnerability-scans/)
### [Enable Docker Content Trust](https://docs.docker.com/engine/security/trust/content_trust/)
### [Configure RBAC in UCP](https://docs.docker.com/datacenter/ucp/2.2/guides/access-control/)
### [Integrate UCP with LDAP/AD](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/configure/external-auth/)
### [Demonstrate creation of UCP client bundles](https://blog.docker.com/2017/09/get-familiar-docker-enterprise-edition-client-bundles/)
### [Describe default engine security](https://docs.docker.com/engine/security/security/)
### [Describe swarm default security](https://docs.docker.com/engine/swarm/how-swarm-mode-works/pki/)
### [Describe MTLS](https://diogomonica.com/2017/01/11/hitless-tls-certificate-rotation-in-go/)
### [Identity roles](https://docs.docker.com/datacenter/ucp/2.2/guides/access-control/permission-levels/#roles)
### [Describe the difference between UCP workers and managers](https://docs.docker.com/datacenter/ucp/2.2/guides/architecture/)
- Describe process to use external certificates with UCP and DTR (**UCP** [from cli](https://success.docker.com/article/how-do-i-provide-an-externally-generated-security-certificate-during-the-ucp-command-line-installation), [from GUI](https://docs.docker.com/ee/ucp/admin/configure/use-your-own-tls-certificates/#configure-ucp-to-use-your-own-tls-certificates-and-keys), [print the public certificates](https://docs.docker.com/datacenter/ucp/3.0/reference/cli/dump-certs/)), [**DTR**](https://docs.docker.com/ee/dtr/admin/configure/use-your-own-tls-certificates/))

## Domain 6: Storage and Volumes (10% of exam)

### [State which graph driver should be used on which OS](https://docs.docker.com/storage/storagedriver/select-storage-driver/)
### [Demonstrate how to configure devicemapper](https://docs.docker.com/storage/storagedriver/device-mapper-driver/#configure-docker-with-the-devicemapper-storage-driver)
### [Compare object storage to block storage, and explain which one is preferable when
  available](https://rancher.com/block-object-file-storage-containers/)
### [Summarize how an application is composed of layers and where those layers reside on
  the filesystem](https://docs.docker.com/storage/storagedriver/#images-and-layers)
### [Describe how volumes are used with Docker for persistent storage](https://docs.docker.com/storage/volumes/)
- Identify the steps you would take to clean up unused images on a filesystem, also on DTR.
  ([image prune](https://docs.docker.com/engine/reference/commandline/image_prune/), [system prune](https://docs.docker.com/engine/reference/commandline/system_prune/) and [from DTR](https://docs.docker.com/ee/dtr/user/manage-images/delete-images/))
### [Demonstrate how storage can be used across cluster nodes](https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins)
