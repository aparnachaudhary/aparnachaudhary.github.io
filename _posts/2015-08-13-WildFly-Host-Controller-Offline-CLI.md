---
layout: post
title: WildFly Host Controller Offline CLI
tags: [wildfly,offline-cli,domain]
---

WildFly 10 introduced support for Offline CLI support for domain mode. Following post is an attemp to experiment with this operation mode to configure domain mode.

## Simple CLI Operations


* Connect to WildFly CLI
```bash
sh bin/jboss-cli.sh
```

* Start embedded Host Controller
```bash
embed-host-controller --std-out=echo --domain-config=domain.xml --host-config=host-master.xml
```


```bash

20:36:25,540 INFO  [org.jboss.modules] (AeshProcess: 1) JBoss Modules version 1.4.3.Final
20:36:25,917 INFO  [org.jboss.msc] (AeshProcess: 1) JBoss MSC version 1.2.6.Final
20:36:25,961 INFO  [org.jboss.as] (MSC service thread 1-7) WFLYSRV0049: WildFly Full 10.0.0.Beta1 (WildFly Core 2.0.0.Beta1) starting
20:36:28,211 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'enabled' in the resource at address '/profile=default/subsystem=datasources/data-source=ExampleDS' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
20:36:28,244 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'enabled' in the resource at address '/profile=ha/subsystem=datasources/data-source=ExampleDS' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
20:36:28,259 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'default-stack' in the resource at address '/profile=ha/subsystem=jgroups' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
20:36:28,279 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'enabled' in the resource at address '/profile=full/subsystem=datasources/data-source=ExampleDS' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
20:36:28,308 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'enabled' in the resource at address '/profile=full-ha/subsystem=datasources/data-source=ExampleDS' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
20:36:28,320 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'default-stack' in the resource at address '/profile=full-ha/subsystem=jgroups' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
20:36:28,560 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 10.0.0.Beta1 (WildFly Core 2.0.0.Beta1) (Host Controller) started in 2759ms - Started 25 of 27 services (8 services are lazy, passive or on-demand)
```

```bash

[domain@embedded /] ls -l
ATTRIBUTE                VALUE             TYPE
domain-organization      undefined         STRING
launch-type              DOMAIN            STRING
local-host-name          master            STRING
management-major-version 4                 INT
management-micro-version 0                 INT
management-minor-version 0                 INT
name                     Unnamed Domain    STRING
namespaces               []                OBJECT
process-type             Domain Controller STRING
product-name             WildFly Full      STRING
product-version          10.0.0.Beta1      STRING
release-codename         Kenny             STRING
release-version          2.0.0.Beta1       STRING
schema-locations         []                OBJECT

CHILD                     MIN-OCCURS MAX-OCCURS
core-service              n/a        n/a
deployment                n/a        n/a
deployment-overlay        n/a        n/a
extension                 n/a        n/a
host                      n/a        n/a
interface                 n/a        n/a
management-client-content n/a        n/a
path                      n/a        n/a
profile                   n/a        n/a
server-group              n/a        n/a
socket-binding-group      n/a        n/a
system-property           n/a        n/a
[domain@embedded /]
```

* Change the main server group to HA profile

```bash

/server-group=main-server-group:write-attribute(name=profile, value=ha)
/server-group=main-server-group:write-attribute(name=socket-binding-group, value=ha-sockets)
```

* Add Infinispan Cache Container

Add the cache to HA profile.

```bash

/profile=ha/subsystem=infinispan/cache-container=myCacheContainer/:add(default-cache=myCache)
/profile=ha/subsystem=infinispan/cache-container=myCacheContainer/transport=TRANSPORT/:add(lock-timeout=60000)
/profile=ha/subsystem=infinispan/cache-container=myCacheContainer:write-attribute(name=module, value=org.wildfly.clustering.server)
/profile=ha/subsystem=infinispan/cache-container=myCacheContainer/replicated-cache=myCache/:add(mode=ASYNC, statistics-enabled=true)
/profile=ha/subsystem=infinispan/cache-container=myCacheContainer/replicated-cache=myCache/transaction=TRANSACTION/:add(mode=BATCH)
```

And now to the full-ha profile.

```bash

/profile=full-ha/subsystem=infinispan/cache-container=myCacheContainer/:add(default-cache=myCache)
/profile=full-ha/subsystem=infinispan/cache-container=myCacheContainer/transport=TRANSPORT/:add(lock-timeout=60000)
/profile=full-ha/subsystem=infinispan/cache-container=myCacheContainer:write-attribute(name=module, value=org.wildfly.clustering.server)
/profile=full-ha/subsystem=infinispan/cache-container=myCacheContainer/replicated-cache=myCache/:add(mode=ASYNC, statistics-enabled=true)
/profile=full-ha/subsystem=infinispan/cache-container=myCacheContainer/replicated-cache=myCache/transaction=TRANSACTION/:add(mode=BATCH)
```

* TODO: Remove Servers

```bash

/host=master/server-config=server-one:remove
/host=master/server-config=server-two:remove
```

* TODO: Add only one server per Host


## Non-Modular Approach

* Connect to WildFly CLI

```bash

java -jar bin/client/jboss-cli-client.jar
```

* Start Embedded Host Controller

```bash

embed-host-controller --jboss-home=/Users/Aparna/dev/tools/wildfly-10.0.0.Beta1
```
Currently fails with NPE.

