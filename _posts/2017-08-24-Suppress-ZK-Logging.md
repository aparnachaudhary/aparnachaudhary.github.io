---
layout: post
title: Suprress ZooKeeper Logging in Spring Boot
tags: [zookeeper, spring-boot]
---


## Problem:

ZooKeeper constantly logs these messages. Adding `logging.level.org.apache.zookeeper=WARN` in `application.properties` does not suppress these error.


```shell
2017-08-17 12:55:41.951  INFO 1 --- [ XNIO-2 task-24] o.a.h.h.zookeeper.RecoverableZooKeeper   : Process identifier=hconnection-0x187edfa9 connecting to ZooKeeper ensemble=localhost12181
2017-08-17 12:55:41.951  INFO 1 --- [ XNIO-2 task-24] org.apache.zookeeper.ZooKeeper           : Initiating client connection, connectString=localhost:12181 sessionTimeout=60000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@31b0624c
2017-08-17 12:55:41.954  INFO 1 --- [localhost:12181)] org.apache.zookeeper.ClientCnxn          : Opening socket connection to server localhost:12181. Will not attempt to authenticate using SASL (unknown error)
2017-08-17 12:55:41.955  INFO 1 --- [localhost:12181)] org.apache.zookeeper.ClientCnxn          : Socket connection established to localhost:12181, initiating session
2017-08-17 12:55:41.957  INFO 1 --- [localhost:12181)] org.apache.zookeeper.ClientCnxn          : Session establishment complete on server localhost:12181, sessionid = 0x15deb0a0218257e, negotiated timeout = 60000
2017-08-17 12:55:41.962  INFO 1 --- [ XNIO-2 task-24] nectionManager$HConnectionImplementation : Closing master protocol: MasterService
2017-08-17 12:55:41.962  INFO 1 --- [ XNIO-2 task-24] nectionManager$HConnectionImplementation : Closing zookeeper sessionid=0x15deb0a0218257e
2017-08-17 12:55:41.965  INFO 1 --- [ XNIO-2 task-24] org.apache.zookeeper.ZooKeeper           : Session: 0x15deb0a0218257e closed
2017-08-17 12:55:41.965  INFO 1 --- [-24-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down
```

## Solution:

Add a `logback.xml` file on the classpath to solve this.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="org.springframework" level="INFO"/>
    <logger name="org.apache.zookeeper" level="WARN"/>
</configuration>
```
