---
title : "Notes for Cluster and Cloud Computing(COMP90024)"
date: 2019-04-17T01:37:56+08:00
lastmod: 2019-04-17T01:37:56+08:00
draft: false
tags: ["cluster", "auto-deploying","distributed"]
categories: ["cluster computing related"]
author: "Yang Zhang"
---
## Information Session

**Cloud Characteristics:**

- **On-demand self-service**: A consumer can provision computing capabilities as needed without requiring human interaction with each service provider.
- **Networked access**: Capabilities are available over the network and accessed through standard mechanisms that promote use by hetergeneous client platforms.
- **Resource pooling**: The provider's computing resources are pooled to serve multiple consumers using a multi-tenant model potentially with different physical and virtual resources that can be dynamically assigned and reassigned according to consumer demand.
- **Rapid elasticity**: Capabilities can be elastically provisioned and released, in some cases automatically, to scale rapidly upon demand.
- **Measured service**: Cloud systems automatically control and optimize resource use by leveraging a metering capability at some level of abstraction appropriate to the type of service. 

**Cloud Computing Flavours:**

- Compute Clouds:
  - Amazon Elastic Compute Cloud
  - Azure
- Data Clouds:
  - Amazon Simple Storage Service
  - Google docs
  - iCloud
  - Dropbox
- Application Clouds:
  - App Store
  - Virtual Image Factories
  - Community-specific
- Public/Private/Hybrid/Mobile/Health Clouds

### History

#### Distributed System
- The first focus is on **Transparency** and **Heterogeneity** of computer-computer interactions. (Finding -> Binding -> checking -> invoking) in heterogeneous environment.

#### Grid Computing
- Grid Computing transfer computer-computer focus to organization-organization focus. 
- Grid computing is distinguished from conventional high-performance computing systems such as cluster computing in that grid computers have each node set to perform a different task/application. Grid computers also tend to be more heterogeneous and geographically dispersed (thus not physically coupled) than cluster computers.
- Although a single grid can be dedicated to a particular application, commonly a grid is used for a variety of purposes. Grids are often constructed with general-purpose grid middleware software libraries. Grid sizes can be quite large.

## Domain Driver


## Parallel System, Distributed Computing and HPC/HTC
## HPC
## Cloud Computing
## ReST and Docker(Containalization)
## Big Data and Related Technologies
## Big Data Analytics
## Cloud Underpinning and Other Things
## Security and Clouds

### Auto-deploying by Ansible
### Docker
#### Background Introduction
##### Virtualization vs. Containerization
The advantages of virtualization, such as application containment and horizontal scalability come at a cost: resources.

The containerization allows virtual instances to share a single host OS to reduce wasted resources since each container only holds the application and related binaries. The rest are shared among the containers.
![Biased Locking](/media/posts/vmcon.png)
##### Container
Container is similar to concept of resouce isolation and allocation as a virtual machine.

Without bunding the entire hardware environment and full OS.

##### Container Orchestration Tools
Container orchestration technologies provides a framework for integrating and managing container at scale.
- Features:
  - Networking
  - Scaling
  - Service discovery and load balancing
  - Health check and self-healing
  - Security  
  - Rolling updates
- Goals:
  - Simplify container management processes
  - Help to manage availability and scaling of containers
#### Introduction to Docker
Docker is by far the most successful containerization technology. It uses resource isolation features of the Linux kernel to allow independent "containers" to run within a single Linux instance.
##### Concepts
- **Container**:a process that behaves like an independent machine, it is a runtime instance of a docker image.
- **Image**: a blueprint for a container.
- **Dockerfile**: the recipe to create an  image
- **Registry**: a hosted service containing repositories of images. (Docker Hub)[https://hub.docker.com]
- **Tag**: a label applied to a Docker image in a repository.
- **Docker Compose**: Compose is a tool for defining and running multi-containers Docker applications.
- **Docker SWARM**: a standalone native clutering / orchestration tool for Docker.
##### Manage Data in Docker
By default, data inside a Docker container won't be persisted when a container is no longer exist.
Docker provides two options for containers to store files on the host machine so that the files are persisted even after the container stops.
- Docker volumes(Managed by Docker, /var/lib/docker/volume/)
- Bind mounts(Managed by user, anywhere on the file system)
![Docker data management](/media/posts/dockermount.png)

##### Networking
- Network mode "host": every container uses the host network stack(share the same IP address). Ports cannot be shared across container(Linux only)
- Network mode "bridge": container can re-use the same port, as they have different IP addresses, and expose a port of thier own that belongs to the hosts, allowing the container to be somewhat visible from the outside.

##### Extra
Normal Web Service requires multiple dependencies such as PHP, MySQL, Redis, Node, etc. Ususally we will install the dependencies manually, which however introduces several problems.

- Different services require the same software with different versions.
- Different services in a single environment will modify the file at the same time, such as config of system, nginx
- The same service needs to be deploy manually, which greatly waste the resources.

The following solution is to pack the whole system, which introduces the virtualization technology. It saves the resources but there are lots of other problems.
- Packed virtual machine file including image is pretty large.
- Packed virtual machine file 
- The process of pack could not be automatic.


### CouchDB