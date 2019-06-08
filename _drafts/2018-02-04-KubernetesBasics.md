---
layout: post
title: Kubernetes Basics
tags: [container-orchestration, kubernetes, docker, containers, k8s]
---

Following blog post will cover some of the concepts related to Kubernetes.

Why Containers?
---------------

* With container image we can bundle application along with its dependencies and runtime.
* Containers can be deployed on platform of choice, desktops, VM's, Cloud, etc

Container Orchsestration?
-------------------------

* Container orchestrators are the tools that groups hosts together to form a cluster.
* Applications need to be fault-tolerant, scalable, use resources optimally,
can discover other applications and communucate, accessible to outside world and can be upgraded/downgraded with Zero Downtime.
* Options are - Docker Swarm, Apache Mesos, Kubernetes, Amazon ECS, Hashicorp Nomad
* Container Orchestrators provide following features:
  * Bring multiple hosts together to forma cluster
  * Schedule containers to run on different hosts
  * Help containers running on one host to communicate with containers from another host
  * Bind container and storage together
  * Bind containers to a higher level construct like services so that we don't have to deal with individual containers
  * Keep resource usage in check and optimize if necessary
  * Allows secure access to applications running inside containers
