---
layout: post
title: JAX-RS Resource and use of @Path
tags: [jaxrs,java, resteasy]
---
Based on JAX-RS specification; +@Path+ annotation is not mandatory on Resource classes. Apparently many implementations seem to deviate from the specification.
_Shouldn't reference implementation adhere to the specification?_

## JAX-RS 2.0 spec:

Resource classes are POJOs that have at least one method annotated with +@Path+ or a request method designator.

## RESTEasy:

### 2.3.5

http://docs.jboss.org/resteasy/docs/2.3.5.Final/userguide/html/Using_Path.html[UserDoc] The +@javax.ws.rs.Path+ annotation must exist on either the class and/or a resource method.

Implementation is broken in jboss-as-7.1.1.Final. +@Path+ is mandatory at class level.

### 3.0.6

http://docs.jboss.org/resteasy/docs/3.0.6.Final/userguide/html/Using_Path.html[UserDoc] The +@javax.ws.rs.Path+ annotation must exist on either the class and/or a resource method.

Works as documented with wildfly-8.0.0.Final.

But seems to be broken with Tomcat 7. +@Path+ is mandatory at class level. https://issues.jboss.org/browse/RESTEASY-1017[RESTEASY-1017]


## Jersey - JAX-RS Reference implementation:

https://jersey.java.net/documentation/latest/jaxrs-resources.html[Documentation] Root resource classes are POJOs (Plain Old Java Objects) *that are annotated with @Path* have at least one method annotated with +@Path+ or a resource method designator annotation such as +@GET+, +@PUT+, +@POST+, +@DELETE+.
