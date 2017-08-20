---
layout: post
title: Docker for Newbies
tags: [wildfly,javaee,docker]
---

Start WildFly on port 8080 and enable port-forwarding to 8180 on host machine.

```bash
docker run -d -i -t -p 8180:8080 jboss/wildfly
```

See running docker processes and access WildFly at http://192.168.59.103:8180/

```bash

bash-3.2$ docker ps
CONTAINER ID        IMAGE                  COMMAND                CREATED             STATUS              PORTS                              NAMES
8217a58de4d0        jboss/wildfly:latest   "/opt/jboss/wildfly/   51 seconds ago      Up 51 seconds       9990/tcp, 0.0.0.0:8180->8080/tcp   loving_perlman
```

Start another WildFly on port 8080 and enable port-forwarding to 8190 on host machine.

```bash

docker run -d -i -t -p 8190:8080 jboss/wildfly
```


See running docker processes and access WildFly at http://192.168.59.103:8190/

```bash

bash-3.2$ docker ps
CONTAINER ID        IMAGE                  COMMAND                CREATED             STATUS              PORTS                              NAMES
d3d7601fb749        jboss/wildfly:latest   "/opt/jboss/wildfly/   26 seconds ago      Up 26 seconds       9990/tcp, 0.0.0.0:8190->8080/tcp   berserk_mestorf
8217a58de4d0        jboss/wildfly:latest   "/opt/jboss/wildfly/   51 seconds ago      Up 51 seconds       9990/tcp, 0.0.0.0:8180->8080/tcp   loving_perlman
```

Stop WildFly instances

```bash

docker stop 8217a58de4d0
docker stop d3d7601fb749
```

Enlist available docker containers

```bash

docker ps -a
```

## Newbie Questions


1. Can I start multiple containers with boot2docker?
        Yes you can.
2. What happens to volume when you commit docker image?
        Volumes are not managed by docker. So committing an image does not commit data from volume.
        
3. I'm getting error "Are you trying to connect to a TLS-enabled daemon without TLS". How to solve?
        Instead of executing _boot2docker_ use _$(/usr/local/bin/boot2docker shellinit)_.

4. How can I control the memory available to a container?
        Use _-m_ flag. E.g. _docker run -i -t -p 8190:8080 -m 256m jboss/wildfly_

In general, it is quite cool that with docker I do not have to copy the WildFly distro to start multiple server processes. 
Looking forward to more fun with docker.
