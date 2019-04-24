---
title : "Notes for distributed system preparation(COMP90024)"
date: 2019-04-17T01:37:56+08:00
lastmod: 2019-04-17T01:37:56+08:00
draft: false
tags: ["cluster", "auto-deploying","distributed"]
categories: ["cluster computing related"]
author: "Yang Zhang"
---
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