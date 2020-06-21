---
title: k8s Note
date: 2020-06-17 21:00:10
tags: k8s
toc: true
---

# Kubernetes Architecture

![k8s_architecture](/photos/k8s-note/k8s_architecture.png)

## Master Node

![master_node](/photos/k8s-note/master_node.png)

control plane responsible for managing the state of a Kubernetes cluster(brain)

users send requests to the master node via a Command Line Interface (CLI) tool, a Web User-Interface (Web UI) Dashboard, or Application Programming Interface (API)

master node replicas are added to the cluster, configured in High-Availability (HA) mode to ensure the control plane's fault tolerance. While only one of the master node replicas actively manages the cluster, the control plane components stay in sync across the master node replicas

4 components:

1. API server
2. Scheduler
3. Controller managers
4. etcd

### API Server

a central control plane component, coordinating all the administrative tasks

reads the Kubernetes cluster's current state from the etcd, and after a call's execution, saves the resulting state of the Kubernetes cluster in etcd(only master plane component talks to etcd)

designed to scale horizontally: it scales by deploying more instances to balance traffic between those instances

highly configurable and customizable, supports the addition of custom API servers

### Scheduler

assign new objects, such as pods, to nodes, based on current Kubernetes cluster state and new object's requirements

takes into account: individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.

highly configurable and customizable, supports additional custom schedulers (specify the name of the custom scheduler in object 's configuration data)

### Controller Managers

run controllers to regulate the state of the Kubernetes cluster

controllers are watch-loops continuously running and comparing the cluster's desired state (provided by objects' configuration data) with its current state (obtained from etcd data store via the API server), includes:

- Node controller: Responsible for noticing and responding when nodes go down.
- Replication controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
- Endpoints controller: Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token controllers: Create default accounts and API access tokens for new namespaces.

corrective action is taken in the cluster until its current state matches the desired state.

- kube-controller-manager: runs controllers responsible to act when nodes become unavailable, to ensure pod counts are as expected, to create endpoints, service accounts, and API access tokens.

- cloud-controller-manager: runs controllers responsible to interact with the underlying infrastructure of a cloud provider when nodes become unavailable, to manage storage volumes when provided by a cloud service, and to manage load balancing and routing.

    - Node controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
    - Route controller: For setting up routes in the underlying cloud infrastructure
    - Service controller: For creating, updating and deleting cloud provider load balancers

### [etcd](https://etcd.io/docs/)

a distributed key-value store which holds cluster state related data, to persist the Kubernetes cluster's state

either configured on the master node (stacked) or on its dedicated host (external)

- when stacked: HA master node replicas ensure etcd resiliency.
- when external: etcd hosts have to be separately replicated for HA mode configuration.

based on [Raft Consensus Algorithm](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14), written in Golang, besides storing the cluster state, also used to store configuration details such as subnets, ConfigMaps, Secrets, etc.

## Worker Node

![worker_node](/photos/k8s-note/worker_node.png)

provides a running environment for client applications, which are encapsulated in Pods, controlled by the cluster control plane agents running on the master node

Pods are scheduled on worker nodes, where they find required compute, memory and storage resources to run, and networking to talk to each other and the outside world.

A Pod is the smallest scheduling unit in Kubernetes, a logical collection of one or more containers scheduled together.

to access the applications from the external world, we connect to worker nodes and not to the master node.

4 parts:

1. Container runtime
2. kubelet
3. kube-proxy
4. Addons for DNS, Dashboard, cluster-level monitoring and logging.

### Container Runtime

responsible for running containers, e.x. Docker, containerd, CRI-O

### Kubelet

an agent running on each node, communicates with the control plane components from the master node, makes sure containers are running in a Pod.

receives Pod definitions, primarily from the API server, and interacts with the container runtime on the node to run containers associated with the Pod.

monitors the health of the Pod's running container

connects to the container runtime using Container Runtime Interface (CRI). which consists of protocol buffers, gRPC API, and libraries.

![CRI](/photos/k8s-note/CRI.png)

CRI implements two services:

- ImageService: responsible for all the image-related operations
- RuntimeService: responsible for all the Pod and container-related operations.

some examples of CRI shims:

- dockershim: with dockershim, containers are created using Docker installed on the worker nodes. Internally, Docker uses containerd to create and manage containers.

![dockershim](/photos/k8s-note/dockershim.png)

- cri-containerd: with cri-containerd, we can directly use Docker's smaller offspring containerd to create and manage containers.

![cri-containerd](/photos/k8s-note/cri-containerd.png)

- cri-o: cri-o enables using any Open Container Initiative (OCI) compatible runtimes with Kubernetes. At the time this course was created, CRI-O supported runC and Clear Containers as container runtimes. However, in principle, any OCI-compliant runtime can be plugged-in.

![cri-o](/photos/k8s-note/cri-o.png)

### Kube-proxy

network agent which runs on each node, responsible for dynamic updates and maintenance of all networking rules on the node, abstracts the details of Pods networking and forwards connection requests to Pods.

implements part of the Kubernetes Service concept.

maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

uses the operating system packet filtering layer if there is one and it's available. Otherwise, forwards the traffic itself.

### Addons

use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features and functionality not yet available in Kubernetes, therefore implemented through 3rd-party pods and services.

- DNS: cluster DNS is a DNS server for Kubernetes services, required to assign DNS records to Kubernetes objects and resources
- Web UI(Dashboard): a general purposed web-based user interface for cluster management
- Container Resource Monitoring: records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.
- Cluster-level Logging: responsible for saving container logs to a central log store with search/browsing interface.

## Networking Challenges

### Container-to-Container Communication Inside Pods

When a Pod is started, a network namespace is created inside the Pod, and all containers running inside the Pod will share that network namespace so that they can talk to each other via localhost.

### Pod-to-Pod Communication Across Nodes

Kubernetes network model "IP-per-Pod"

containers are integrated with the overall Kubernetes networking model through the use of the [Container Network Interface](https://github.com/containernetworking/cni) (CNI) supported by [CNI plugins](https://github.com/containernetworking/cni#3rd-party-plugins).

### Pod-to-External World Communication

by services, complex constructs which encapsulate networking rules definitions on cluster nodes. By exposing services to the external world with kube-proxy, applications become accessible from outside the cluster over a virtual IP.

# Installing Kubernetes

## Kubernetes Configuration

four major installation types:

- All-in-One Single-Node Installation:

In this setup, all the master and worker components are installed and running on a single-node. While it is useful for learning, development, and testing, it should not be used in production. Minikube is one such example, and we are going to explore it in future chapters.

- Single-Node etcd, Single-Master and Multi-Worker Installation:

In this setup, we have a single-master node, which also runs a single-node etcd instance. Multiple worker nodes are connected to the master node.

- Single-Node etcd, Multi-Master and Multi-Worker Installation:

In this setup, we have multiple-master nodes configured in HA mode, but we have a single-node etcd instance. Multiple worker nodes are connected to the master nodes.

- Multi-Node etcd, Multi-Master and Multi-Worker Installation:

In this mode, etcd is configured in clustered HA mode, the master nodes are all configured in HA mode, connecting to multiple worker nodes. This is the most advanced and recommended production setup.

## Localhost Installation

localhost installation options available to deploy single- or multi-node Kubernetes clusters on our workstation/laptop:

- Minikube: single-node local Kubernetes cluster
- Docker Desktop: single-node local Kubernetes cluster for Windows and Mac
- CDK on LXD: multi-node local cluster with LXD containers.

## On-Premise Installation

Kubernetes can be installed on-premise on VMs and bare metal.

- On-Premise VMs:

Kubernetes can be installed on VMs created via Vagrant, VMware vSphere, KVM, or another Configuration Management (CM) tool in conjunction with a hypervisor software. There are different tools available to automate the installation, such as Ansible or kubeadm.

- On-Premise Bare Metal:

Kubernetes can be installed on on-premise bare metal, on top of different operating systems, like RHEL, CoreOS, CentOS, Fedora, Ubuntu, etc. Most of the tools used to install Kubernetes on VMs can be used with bare metal installations as well.

## Cloud Installation

Kubernetes can be installed and managed on almost any cloud environment:

- Hosted Solutions:

With Hosted Solutions, any given software is completely managed by the provider. The user pays hosting and management charges. Some of the vendors providing hosted solutions for Kubernetes are:

> Google Kubernetes Engine (GKE)
> Azure Kubernetes Service (AKS)
> Amazon Elastic Container Service for Kubernetes (EKS)
> DigitalOcean Kubernetes
> OpenShift Dedicated
> Platform9
> IBM Cloud Kubernetes Service.

- Turnkey Cloud Solutions:

Below are only a few of the Turnkey Cloud Solutions, to install Kubernetes with just a few commands on an underlying IaaS platform, such as:

> Google Compute Engine (GCE)
> Amazon AWS (AWS EC2)
> Microsoft Azure (AKS).

- Turnkey On-Premise Solutions:

The On-Premise Solutions install Kubernetes on secure internal private clouds with just a few commands:

> GKE On-Prem by Google Cloud
> IBM Cloud Private
> OpenShift Container Platform by Red Hat.

## Kubernetes Installation Tools/Resources

It is worth checking out the [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) GitHub project

some useful tools/resources available:

- kubeadm:

kubeadm is a first-class citizen on the Kubernetes ecosystem. It is a secure and recommended way to bootstrap a single- or multi-node Kubernetes cluster. It has a set of building blocks to setup the cluster, but it is easily extendable to add more features. Please note that kubeadm does not support the provisioning of hosts.

- kubespray

With kubespray (formerly known as kargo), we can install Highly Available Kubernetes clusters on AWS, GCE, Azure, OpenStack, or bare metal. Kubespray is based on Ansible, and is available on most Linux distributions. It is a Kubernetes Incubator project.

- kops

With kops, we can create, destroy, upgrade, and maintain production-grade, highly-available Kubernetes clusters from the command line. It can provision the machines as well. Currently, AWS is officially supported. Support for GCE is in beta, and VMware vSphere in alpha stage, and other platforms are planned for the future. Explore the kops project for more details.

- kube-aws

With kube-aws we can create, upgrade and destroy Kubernetes clusters on AWS from the command line. Kube-aws is also a Kubernetes Incubator project.

## Minikube

A Local Single-Node Kubernetes Cluster
