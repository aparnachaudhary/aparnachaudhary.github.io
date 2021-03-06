---
layout: post
title: Keep your WAR/EAR files clean with jQAssistant
tags: [wildfly,javaee,jqassistant]
---

JavaEE specification JARs and installed libraries shall not be packaged inside EAR/WAR files, as they are provided by the container.
But not all containers result in deployment failure when they are bundled inside EAR/WAR files.

Quote from JSR-342 Java Platform, Enterprise Edition 7 Specification:


> EE.8.2.3 Library Conflicts
> If an application includes a bundled version of a library, and the same library exists as an installed library, 
> the instance of the library bundled with the application should be used in preference to any installed version of the > library. 
> This allows an application to bundle exactly the version of a library it requires without being influenced by any installed libraries. 
> Note that if the library is also a required component of the Java EE platform version on which the application is being deployed, the platform version may (and typically will) take precedence.

Typically dependency management becomes an after thought in enterprise projects:

* When size of deliverable increases
* Refactoring becomes difficult
* Class Loading Hell

If you are using maven; solution to this problem is to use the right scopes. But to err is human.
So the question is how can we keep the EAR files lean and mean? http://jqassistant.org/[jQAssistant] to the Rescue!


## How to identify bloated EAR/WAR files

### Scan EAR/WAR files using jQAssistant

Follow jQAssistant http://jqassistant.org/get-started/[Getting Started guide]


```cypher
bin/jqassistant.sh scan -f wildfly-kitchensink-ear-ear.ear
```

### Inspect content of the EAR/WAR

Following cypher query can be used to find EAR files that bundles EJB spec.

```cypher
MATCH
  (ear:Enterprise:Application:Archive)-[:CONTAINS]->(jar:File:Java:Jar:Archive)
WHERE
  jar.fileName =~ ".*jboss-ejb-api_3.2_spec.jar"
RETURN
  ear.fileName
```

Following cypher query can be used to find WAR files that bundle JSF spec.

```cypher

MATCH 
  (war:Web:Application:Archive)-[:CONTAINS]->(jar:File:Java:Jar:Archive)
WHERE 
  jar.fileName =~ ".*jboss-jsf-api_2.2_spec-2.2.8.jar"
RETURN
  war.fileName
```

## Resources

* https://docs.jboss.org/author/display/WFLY9/Class+Loading+in+WildFly
* http://www.oracle.com/au/products/database/packaging-best-practices-090628.html
