---
layout: post
title: Sharing client libraries in Microservices
tags: [microservices, shared-lib, trade-off]
---

So what's bothering me lately is the question - Shared client libraries in Microservices - is it good or bad or it depends ;)


PROS
----
* Code Reuse
* Defects can be fixed at single location

CONS
----

* One of the main benefits of microservices is independence; in terms of language, code, team. Shared client library increases co-ordination.
* With library comes transitive dependencies - AaHa... - so you are potentially stuck with HttpClient 3.x.
* Mental Barrier to make changes to shared libraries.



References
----------

* https://blog.philipphauer.de/dont-share-libraries-among-microservices/
* 
