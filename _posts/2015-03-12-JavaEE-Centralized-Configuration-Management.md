---
layout: post
title: JavaEE - Centralized Configuration Management
tags: [wildfly,javaee,configuration]
---

When it comes to application configuration; there is no golden solution - it typically depends on your development and deployment environment.
A Configuration JSR was proposed but it is not part of JavaEE yet - http://javaeeconfig.blogspot.nl/2014/05/java-ee-config-feature-details.html.

TIP: If you are using JBossAS/WildFly; then this post is a good starting point https://community.jboss.org/wiki/HowToPutAnExternalFileInTheClasspath

Following approaches are typically used for application specific configuration management.

## Core Modules and property files

* https://community.jboss.org/wiki/HowToPutAnExternalFileInTheClasspath
* http://blog.jyore.com/?p=58
* http://blog.andyserver.com/2013/10/java-jboss-eap-61-power-to-the-properties/
* https://github.com/sabre1041/jboss-properties


## Naming service (JNDI)

* http://blog.diabol.se/?p=312

## System Properties

* http://blog.andyserver.com/2013/10/java-jboss-eap-61-power-to-the-properties/
* http://blog.chris-ritchie.com/2015/03/inject-external-properties-cdi-java-wildfly.html

## Database configuration

CAUTION: As I said before; there is no golden rule. But storing configuration in database can be undesired especially
if you are thinking about blue-green deployments.
