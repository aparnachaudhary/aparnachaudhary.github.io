---
layout: post
title: WildFly 9.x Modules and versions
tags: [wildfly,javaee,jqassistant]
---

JavaEE specification JARs and installed libraries shall not be packaged inside EAR/WAR files, as they are provided by the container. Following blog post provides an overview of such third jars and libs.


Quote from JSR-342 Java Platform, Enterprise Edition 7 Specification:

[quote, JavaEE Spec]
-------------------------
EE.8.2.3 Library Conflicts
If an application includes a bundled version of a library, and the same library exists as an installed library,
the instance of the library bundled with the application should be used in preference to any installed version of the library.
This allows an application to bundle exactly the version of a library it requires without being influenced by any installed libraries.
Note that if the library is also a required component of the Java EE platform version on which the application is being deployed,
the platform version may (and typically will) take precedence.
-------------------------

Bundling these JAR files inside EAR/WAR results in big fat deployables. But this is not the main concern. Packaging duplicate JARs can result in 
undesired classloading issues which are difficult to diagnose. In addition to this, migration to newer version of WildFly can be painful.

### Scan WildFly modules directory 

Let's scan the module directory using http://jqassistant.org/get-started/[jQAssistant].

```bash
jqassistant.sh scan -f $WILDFLY_HOME/modules/system/layers/base/
```

### Start the Neo4J Server:

```bash

jqassistant.sh server
```

### Query Execution

Finding all the modules in WildFly.

```cypher

MATCH (n:File:Xml)-[:HAS_ROOT_ELEMENT]->(root:Xml:Element{name:"module"}),
root-[HAS_ATTRIBUTE]->(module:Xml:Attribute{name:"name"})
SET n:WildFly:ModuleDesc
RETURN n.fileName,module.value ORDER BY n.fileName
```

Sample output:

```cypher

neo4j-sh (?)$ MATCH (n:File:Xml)-[:HAS_ROOT_ELEMENT]->(root:Xml:Element{name:"module"}), root-[HAS_ATTRIBUTE]->(module:Xml:Attribute{name:"name"}) SET n:WildFly:ModuleDesc RETURN n.fileName,module.value ORDER BY n.fileName;
==> +----------------------------------------------------------------------------------------------------------------------------------------+
==> | n.fileName                                                                 | module.value                                              |
==> +----------------------------------------------------------------------------------------------------------------------------------------+
==> | "/asm/asm/main/module.xml"                                                 | "asm.asm"                                                 |
==> | "/ch/qos/cal10n/main/module.xml"                                           | "ch.qos.cal10n"                                           |
==> | "/com/fasterxml/classmate/main/module.xml"                                 | "com.fasterxml.classmate"                                 |
==> | "/com/fasterxml/jackson/core/jackson-annotations/main/module.xml"          | "com.fasterxml.jackson.core.jackson-annotations"          |
==> | "/com/fasterxml/jackson/core/jackson-core/main/module.xml"                 | "com.fasterxml.jackson.core.jackson-core"                 |
==> | "/com/fasterxml/jackson/core/jackson-databind/main/module.xml"             | "com.fasterxml.jackson.core.jackson-databind"             |
==> | "/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/module.xml" | "com.fasterxml.jackson.jaxrs.jackson-jaxrs-json-provider" |
==> | "/com/github/relaxng/main/module.xml"                                      | "com.github.relaxng"                                      |
==> | "/com/google/guava/main/module.xml"                                        | "com.google.guava"                                        |
==> | "/com/h2database/h2/main/module.xml"                                       | "com.h2database.h2"                                       |
```



Now execute the simple Cypher query to find out all the JAR files that are available in WildFly.

```cypher

MATCH (n:Directory)-[:CONTAINS]->(f:File:Jar) RETURN DISTINCT f.fileName ORDER BY f.fileName
```

=== Result

Example output for the above query [wildfly-9.0.0.Alpha2-SNAPSHOT (build 1601)].

```bash

neo4j-sh (?)$ MATCH (n:Directory)-[:CONTAINS]->(f:File:Jar) RETURN DISTINCT f.fileName ORDER BY f.fileName;
==> +--------------------------------------------------------------------------------------------------------------+
==> | f.fileName                                                                                                   |
==> +--------------------------------------------------------------------------------------------------------------+
==> | "/asm/asm/main/asm-3.3.1.jar"                                                                                |
==> | "/ch/qos/cal10n/main/cal10n-api-0.8.1.jar"                                                                   |
==> | "/com/fasterxml/classmate/main/classmate-1.0.0.jar"                                                          |
==> | "/com/fasterxml/jackson/core/jackson-annotations/main/jackson-annotations-2.4.1.jar"                         |
==> | "/com/fasterxml/jackson/core/jackson-core/main/jackson-core-2.4.1.jar"                                       |
==> | "/com/fasterxml/jackson/core/jackson-databind/main/jackson-databind-2.4.1.jar"                               |
==> | "/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/jackson-jaxrs-base-2.4.1.jar"                 |
==> | "/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/jackson-jaxrs-json-provider-2.4.1.jar"        |
==> | "/com/fasterxml/jackson/jaxrs/jackson-jaxrs-json-provider/main/jackson-module-jaxb-annotations-2.4.1.jar"    |
==> | "/com/github/relaxng/main/relaxngDatatype-2011.1.jar"                                                        |
==> | "/com/google/guava/main/guava-17.0.jar"                                                                      |
==> | "/com/h2database/h2/main/h2-1.3.173.jar"                                                                     |
==> | "/com/sun/codemodel/main/codemodel-2.6.jar"                                                                  |
==> | "/com/sun/istack/main/istack-commons-runtime-2.6.1.jar"                                                      |
==> | "/com/sun/istack/main/istack-commons-tools-2.6.1.jar"                                                        |
==> | "/com/sun/jsf-impl/main/jsf-impl-2.2.10-jbossorg-1.jar"                                                      |
==> | "/com/sun/xml/bind/main/jaxb-core-2.2.7.jar"                                                                 |
==> | "/com/sun/xml/bind/main/jaxb-impl-2.2.7.jar"                                                                 |
==> | "/com/sun/xml/bind/main/jaxb-xjc-2.2.7.jar"                                                                  |
==> | "/com/sun/xml/fastinfoset/main/FastInfoset-1.2.13.jar"                                                       |
==> | "/com/sun/xml/messaging/saaj/main/saaj-impl-1.3.16-jbossorg-1.jar"                                           |
==> | "/com/sun/xml/txw2/main/txw2-20110809.jar"                                                                   |
==> | "/com/sun/xsom/main/xsom-20110809.jar"                                                                       |
==> | "/gnu/getopt/main/java-getopt-1.0.13.jar"                                                                    |
==> | "/io/netty/main/netty-all-4.0.15.Final.jar"                                                                  |
==> | "/io/undertow/core/main/undertow-core-1.2.0.Beta8.jar"                                                       |
==> | "/io/undertow/jsp/main/ecj-4.4.1.jar"                                                                        |
==> | "/io/undertow/jsp/main/jastow-1.1.1.Final.jar"                                                               |
==> | "/io/undertow/servlet/main/undertow-servlet-1.2.0.Beta8.jar"                                                 |
==> | "/io/undertow/websocket/main/undertow-websockets-jsr-1.2.0.Beta8.jar"                                        |
==> | "/javax/activation/api/main/activation-1.1.1.jar"                                                            |
==> | "/javax/annotation/api/main/jboss-annotations-api_1.2_spec-1.0.0.Final.jar"                                  |
==> | "/javax/batch/api/main/jboss-batch-api_1.0_spec-1.0.0.Final.jar"                                             |
==> | "/javax/ejb/api/main/jboss-ejb-api_3.2_spec-1.0.0.Final.jar"                                                 |
==> | "/javax/el/api/main/jboss-el-api_3.0_spec-1.0.4.Final.jar"                                                   |
==> | "/javax/enterprise/api/main/cdi-api-1.2.jar"                                                                 |
==> | "/javax/enterprise/concurrent/api/main/jboss-concurrency-api_1.0_spec-1.0.0.Final.jar"                       |
==> | "/javax/faces/api/main/jboss-jsf-api_2.2_spec-2.2.10.jar"                                                    |
==> | "/javax/inject/api/main/javax.inject-1.jar"                                                                  |
==> | "/javax/interceptor/api/main/jboss-interceptors-api_1.2_spec-1.0.0.Final.jar"                                |
==> | "/javax/jms/api/main/jboss-jms-api_2.0_spec-1.0.0.Final.jar"                                                 |
==> | "/javax/jws/api/main/jsr181-api-1.0-MR1.jar"                                                                 |
==> | "/javax/mail/api/main/javax.mail-1.5.2.jar"                                                                  |
==> | "/javax/management/j2ee/api/main/jboss-j2eemgmt-api_1.1_spec-1.0.1.Final.jar"                                |
==> | "/javax/orb/api/main/openjdk-orb-8.0.2.Beta1.jar"                                                            |
==> | "/javax/persistence/api/main/hibernate-jpa-2.1-api-1.0.0.Final.jar"                                          |
==> | "/javax/resource/api/main/jboss-connector-api_1.7_spec-1.0.0.Final.jar"                                      |
==> | "/javax/security/auth/message/api/main/jboss-jaspi-api_1.1_spec-1.0.0.Final.jar"                             |
==> | "/javax/security/jacc/api/main/jboss-jacc-api_1.5_spec-1.0.0.Final.jar"                                      |
==> | "/javax/servlet/api/main/jboss-servlet-api_3.1_spec-1.0.0.Final.jar"                                         |
==> | "/javax/servlet/jsp/api/main/jboss-jsp-api_2.3_spec-1.0.1.Final.jar"                                         |
==> | "/javax/servlet/jstl/api/main/jboss-jstl-api_1.2_spec-1.1.2.Final.jar"                                       |
==> | "/javax/sql/api/main/jboss-javax-sql-api_7.0_spec-1.0.0.Final.jar"                                           |
==> | "/javax/transaction/api/main/jboss-transaction-api_1.2_spec-1.0.0.Final.jar"                                 |
==> | "/javax/validation/api/main/validation-api-1.1.0.Final.jar"                                                  |
==> | "/javax/websocket/api/main/jboss-websocket-api_1.1_spec-1.1.1.Final.jar"                                     |
==> | "/javax/ws/rs/api/main/jaxrs-api-3.0.10.Final.jar"                                                           |
==> | "/javax/wsdl4j/api/main/wsdl4j-1.6.3.jar"                                                                    |
==> | "/javax/xml/bind/api/main/jboss-jaxb-api_2.2_spec-1.0.4.Final.jar"                                           |
==> | "/javax/xml/rpc/api/main/jboss-jaxrpc-api_1.1_spec-1.0.1.Final.jar"                                          |
==> | "/javax/xml/soap/api/main/jboss-saaj-api_1.3_spec-1.0.3.Final.jar"                                           |
==> | "/javax/xml/ws/api/main/jboss-jaxws-api_2.2_spec-2.0.2.Final.jar"                                            |
==> | "/net/jcip/main/jcip-annotations-1.0.jar"                                                                    |
==> | "/nu/xom/main/xom-1.2.10.jar"                                                                                |
==> | "/org/antlr/main/antlr-2.7.7.jar"                                                                            |
==> | "/org/apache/avro/main/avro-1.7.6.jar"                                                                       |
==> | "/org/apache/commons/beanutils/main/commons-beanutils-core-1.8.3.jar"                                        |
==> | "/org/apache/commons/cli/main/commons-cli-1.2.jar"                                                           |
==> | "/org/apache/commons/codec/main/commons-codec-1.9.jar"                                                       |
==> | "/org/apache/commons/collections/main/commons-collections-3.2.1.jar"                                         |
==> | "/org/apache/commons/configuration/main/commons-configuration-1.6.jar"                                       |
==> | "/org/apache/commons/io/main/commons-io-2.4.jar"                                                             |
==> | "/org/apache/commons/lang/main/commons-lang-2.6.jar"                                                         |
==> | "/org/apache/commons/pool/main/commons-pool-1.6.jar"                                                         |
==> | "/org/apache/cxf/impl/main/cxf-rt-bindings-coloc-3.0.4.jar"                                                  |
==> | "/org/apache/cxf/impl/main/cxf-rt-bindings-object-3.0.4.jar"                                                 |
==> | "/org/apache/cxf/impl/main/cxf-rt-bindings-soap-3.0.4.jar"                                                   |
==> | "/org/apache/cxf/impl/main/cxf-rt-bindings-xml-3.0.4.jar"                                                    |
==> | "/org/apache/cxf/impl/main/cxf-rt-databinding-aegis-3.0.4.jar"                                               |
==> | "/org/apache/cxf/impl/main/cxf-rt-databinding-jaxb-3.0.4.jar"                                                |
==> | "/org/apache/cxf/impl/main/cxf-rt-frontend-jaxws-3.0.4.jar"                                                  |
==> | "/org/apache/cxf/impl/main/cxf-rt-frontend-simple-3.0.4.jar"                                                 |
==> | "/org/apache/cxf/impl/main/cxf-rt-management-3.0.4.jar"                                                      |
==> | "/org/apache/cxf/impl/main/cxf-rt-security-3.0.4-jandex.jar"                                                 |
==> | "/org/apache/cxf/impl/main/cxf-rt-security-3.0.4.jar"                                                        |
==> | "/org/apache/cxf/impl/main/cxf-rt-transports-http-3.0.4.jar"                                                 |
==> | "/org/apache/cxf/impl/main/cxf-rt-transports-http-hc-3.0.4.jar"                                              |
==> | "/org/apache/cxf/impl/main/cxf-rt-transports-jms-3.0.4.jar"                                                  |
==> | "/org/apache/cxf/impl/main/cxf-rt-transports-local-3.0.4.jar"                                                |
==> | "/org/apache/cxf/impl/main/cxf-rt-ws-addr-3.0.4.jar"                                                         |
==> | "/org/apache/cxf/impl/main/cxf-rt-ws-mex-3.0.4.jar"                                                          |
==> | "/org/apache/cxf/impl/main/cxf-rt-ws-policy-3.0.4.jar"                                                       |
==> | "/org/apache/cxf/impl/main/cxf-rt-ws-rm-3.0.4.jar"                                                           |
==> | "/org/apache/cxf/impl/main/cxf-rt-ws-security-3.0.4-jandex.jar"                                              |
==> | "/org/apache/cxf/impl/main/cxf-rt-ws-security-3.0.4.jar"                                                     |
==> | "/org/apache/cxf/impl/main/cxf-rt-wsdl-3.0.4.jar"                                                            |
==> | "/org/apache/cxf/impl/main/cxf-services-sts-core-3.0.4.jar"                                                  |
==> | "/org/apache/cxf/impl/main/cxf-services-ws-discovery-api-3.0.4.jar"                                          |
==> | "/org/apache/cxf/impl/main/cxf-tools-common-3.0.4.jar"                                                       |
==> | "/org/apache/cxf/impl/main/cxf-tools-java2ws-3.0.4.jar"                                                      |
==> | "/org/apache/cxf/impl/main/cxf-tools-validator-3.0.4.jar"                                                    |
==> | "/org/apache/cxf/impl/main/cxf-tools-wsdlto-core-3.0.4.jar"                                                  |
==> | "/org/apache/cxf/impl/main/cxf-tools-wsdlto-databinding-jaxb-3.0.4.jar"                                      |
==> | "/org/apache/cxf/impl/main/cxf-tools-wsdlto-frontend-jaxws-3.0.4.jar"                                        |
==> | "/org/apache/cxf/impl/main/cxf-xjc-boolean-3.0.3.jar"                                                        |
==> | "/org/apache/cxf/impl/main/cxf-xjc-dv-3.0.3.jar"                                                             |
==> | "/org/apache/cxf/impl/main/cxf-xjc-ts-3.0.3.jar"                                                             |
==> | "/org/apache/cxf/main/cxf-core-3.0.4.jar"                                                                    |
==> | "/org/apache/httpcomponents/main/httpasyncclient-4.0.1.jar"                                                  |
==> | "/org/apache/httpcomponents/main/httpclient-4.3.6.jar"                                                       |
==> | "/org/apache/httpcomponents/main/httpcore-4.3.3.jar"                                                         |
==> | "/org/apache/httpcomponents/main/httpcore-nio-4.3.3.jar"                                                     |
==> | "/org/apache/httpcomponents/main/httpmime-4.3.6.jar"                                                         |
==> | "/org/apache/james/mime4j/main/apache-mime4j-0.6.jar"                                                        |
==> | "/org/apache/lucene/main/lucene-analyzers-common-4.10.2.jar"                                                 |
==> | "/org/apache/lucene/main/lucene-core-4.10.2.jar"                                                             |
==> | "/org/apache/lucene/main/lucene-facet-4.10.2.jar"                                                            |
==> | "/org/apache/lucene/main/lucene-memory-4.10.2.jar"                                                           |
==> | "/org/apache/lucene/main/lucene-queries-4.10.2.jar"                                                          |
==> | "/org/apache/neethi/main/neethi-3.0.3.jar"                                                                   |
==> | "/org/apache/openjpa/main/jipijapa-openjpa-1.0.1.Final.jar"                                                  |
==> | "/org/apache/qpid/proton/main/proton-api-0.4.jar"                                                            |
==> | "/org/apache/qpid/proton/main/proton-j-impl-0.4.jar"                                                         |
==> | "/org/apache/qpid/proton/main/proton-jms-0.4.jar"                                                            |
==> | "/org/apache/santuario/xmlsec/main/xmlsec-2.0.3.jar"                                                         |
==> | "/org/apache/velocity/main/velocity-1.7.jar"                                                                 |
==> | "/org/apache/ws/security/main/jasypt-1.9.1.jar"                                                              |
==> | "/org/apache/ws/security/main/wss4j-bindings-2.0.3.jar"                                                      |
==> | "/org/apache/ws/security/main/wss4j-policy-2.0.3.jar"                                                        |
==> | "/org/apache/ws/security/main/wss4j-ws-security-common-2.0.3.jar"                                            |
==> | "/org/apache/ws/security/main/wss4j-ws-security-dom-2.0.3.jar"                                               |
==> | "/org/apache/ws/security/main/wss4j-ws-security-policy-stax-2.0.3.jar"                                       |
==> | "/org/apache/ws/security/main/wss4j-ws-security-stax-2.0.3.jar"                                              |
==> | "/org/apache/ws/xmlschema/main/xmlschema-core-2.0.2.jar"                                                     |
==> | "/org/apache/xalan/main/serializer-2.7.1.jbossorg-1.jar"                                                     |
==> | "/org/apache/xalan/main/xalan-2.7.1.jbossorg-1.jar"                                                          |
==> | "/org/apache/xerces/main/xercesImpl-2.9.1-jbossas-2.jar"                                                     |
==> | "/org/apache/xml-resolver/main/xml-resolver-1.2.jar"                                                         |
==> | "/org/bouncycastle/main/bcmail-jdk15on-1.50.jar"                                                             |
==> | "/org/bouncycastle/main/bcpkix-jdk15on-1.50.jar"                                                             |
==> | "/org/bouncycastle/main/bcprov-jdk15on-1.50.jar"                                                             |
==> | "/org/codehaus/jackson/jackson-core-asl/main/jackson-core-asl-1.9.13.jar"                                    |
==> | "/org/codehaus/jackson/jackson-jaxrs/main/jackson-jaxrs-1.9.13.jar"                                          |
==> | "/org/codehaus/jackson/jackson-mapper-asl/main/jackson-mapper-asl-1.9.13.jar"                                |
==> | "/org/codehaus/jackson/jackson-xc/main/jackson-xc-1.9.13.jar"                                                |
==> | "/org/codehaus/jettison/main/jettison-1.3.1.jar"                                                             |
==> | "/org/codehaus/woodstox/main/stax2-api-3.1.4.jar"                                                            |
==> | "/org/codehaus/woodstox/main/woodstox-core-asl-4.4.1.jar"                                                    |
==> | "/org/dom4j/main/dom4j-1.6.1.jar"                                                                            |
==> | "/org/eclipse/persistence/main/jipijapa-eclipselink-1.0.1.Final.jar"                                         |
==> | "/org/fusesource/jansi/main/jansi-1.9.jar"                                                                   |
==> | "/org/glassfish/javax/el/main/javax.el-impl-3.0.1-b05-jbossorg-1.jar"                                        |
==> | "/org/glassfish/javax/enterprise/concurrent/main/javax.enterprise.concurrent-1.0.jar"                        |
==> | "/org/glassfish/javax/json/main/javax.json-1.0.3.jar"                                                        |
==> | "/org/hibernate/3/jipijapa-hibernate3-1.0.1.Final.jar"                                                       |
==> | "/org/hibernate/4.1/jipijapa-hibernate4-1-1.0.1.Final.jar"                                                   |
==> | "/org/hibernate/commons-annotations/main/hibernate-commons-annotations-4.0.5.Final.jar"                      |
==> | "/org/hibernate/hql/main/hibernate-hql-lucene-1.1.0.Final.jar"                                               |
==> | "/org/hibernate/hql/main/hibernate-hql-parser-1.1.0.Final.jar"                                               |
==> | "/org/hibernate/main/hibernate-core-4.3.8.Final.jar"                                                         |
==> | "/org/hibernate/main/hibernate-entitymanager-4.3.8.Final.jar"                                                |
==> | "/org/hibernate/main/hibernate-envers-4.3.8.Final.jar"                                                       |
==> | "/org/hibernate/main/hibernate-infinispan-4.3.8.Final.jar"                                                   |
==> | "/org/hibernate/main/jipijapa-hibernate4-3-1.0.1.Final.jar"                                                  |
==> | "/org/hibernate/search/engine/main/hibernate-search-engine-5.0.0.Final.jar"                                  |
==> | "/org/hibernate/search/engine/main/hibernate-search-infinispan-5.0.0.Final.jar"                              |
==> | "/org/hibernate/search/orm/main/hibernate-search-orm-5.0.0.Final.jar"                                        |
==> | "/org/hibernate/validator/cdi/main/hibernate-validator-cdi-5.1.3.Final.jar"                                  |
==> | "/org/hibernate/validator/main/hibernate-validator-5.1.3.Final.jar"                                          |
==> | "/org/hornetq/main/hornetq-commons-2.4.5.Final.jar"                                                          |
==> | "/org/hornetq/main/hornetq-core-client-2.4.5.Final.jar"                                                      |
==> | "/org/hornetq/main/hornetq-jms-client-2.4.5.Final.jar"                                                       |
==> | "/org/hornetq/main/hornetq-jms-server-2.4.5.Final.jar"                                                       |
==> | "/org/hornetq/main/hornetq-journal-2.4.5.Final.jar"                                                          |
==> | "/org/hornetq/main/hornetq-native-2.4.5.Final.jar"                                                           |
==> | "/org/hornetq/main/hornetq-server-2.4.5.Final.jar"                                                           |
==> | "/org/hornetq/main/hornetq-tools-2.4.5.Final.jar"                                                            |
==> | "/org/hornetq/protocol/amqp/main/hornetq-amqp-protocol-2.4.5.Final.jar"                                      |
==> | "/org/hornetq/protocol/stomp/main/hornetq-stomp-protocol-2.4.5.Final.jar"                                    |
==> | "/org/hornetq/ra/main/hornetq-ra-2.4.5.Final.jar"                                                            |
==> | "/org/infinispan/cachestore/jdbc/main/infinispan-cachestore-jdbc-7.1.1.Final.jar"                            |
==> | "/org/infinispan/cachestore/remote/main/infinispan-cachestore-remote-7.1.1.Final.jar"                        |
==> | "/org/infinispan/client/hotrod/main/infinispan-client-hotrod-7.1.1.Final.jar"                                |
==> | "/org/infinispan/commons/main/infinispan-commons-7.1.1.Final.jar"                                            |
==> | "/org/infinispan/lucene/directory/main/infinispan-lucene-directory-7.1.1.Final.jar"                          |
==> | "/org/infinispan/main/infinispan-core-7.1.1.Final.jar"                                                       |
==> | "/org/infinispan/query/dsl/main/infinispan-query-dsl-7.1.1.Final.jar"                                        |
==> | "/org/infinispan/query/main/infinispan-objectfilter-7.1.1.Final.jar"                                         |
==> | "/org/infinispan/query/main/infinispan-query-7.1.1.Final.jar"                                                |
==> | "/org/javassist/main/javassist-3.18.1-GA.jar"                                                                |
==> | "/org/jaxen/main/jaxen-1.1.3.jar"                                                                            |
==> | "/org/jberet/jberet-core/main/jberet-core-1.1.0.Beta1.jar"                                                   |
==> | "/org/jboss/aesh/main/aesh-0.33.14.jar"                                                                      |
==> | "/org/jboss/as/appclient/main/wildfly-appclient-9.0.0.Alpha2-SNAPSHOT.jar"                                   |
==> | "/org/jboss/as/cli/main/wildfly-cli-1.0.0.Alpha19.jar"                                                       |
==> | "/org/jboss/as/clustering/common/main/wildfly-clustering-common-9.0.0.Alpha2-SNAPSHOT.jar"                   |
==> | "/org/jboss/as/clustering/infinispan/main/wildfly-clustering-infinispan-extension-9.0.0.Alpha2-SNAPSHOT.jar" |
==> | "/org/jboss/as/clustering/jgroups/main/wildfly-clustering-jgroups-extension-9.0.0.Alpha2-SNAPSHOT.jar"       |
==> | "/org/jboss/as/cmp/main/wildfly-cmp-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/as/connector/main/wildfly-connector-9.0.0.Alpha2-SNAPSHOT.jar"                                   |
==> | "/org/jboss/as/console/main/release-stream-2.5.5.Final-resources.jar"                                        |
==> | "/org/jboss/as/controller-client/main/wildfly-controller-client-1.0.0.Alpha19.jar"                           |
==> | "/org/jboss/as/controller/main/wildfly-controller-1.0.0.Alpha19.jar"                                         |
==> | "/org/jboss/as/core-security-api/main/wildfly-core-security-api-1.0.0.Alpha19.jar"                           |
==> | "/org/jboss/as/core-security/main/wildfly-core-security-1.0.0.Alpha19.jar"                                   |
==> | "/org/jboss/as/deployment-repository/main/wildfly-deployment-repository-1.0.0.Alpha19.jar"                   |
==> | "/org/jboss/as/deployment-scanner/main/wildfly-deployment-scanner-1.0.0.Alpha19.jar"                         |
==> | "/org/jboss/as/domain-http-error-context/main/wildfly-domain-http-error-context-1.0.0.Alpha19.jar"           |
==> | "/org/jboss/as/domain-http-interface/main/wildfly-domain-http-interface-1.0.0.Alpha19.jar"                   |
==> | "/org/jboss/as/domain-management/main/wildfly-domain-management-1.0.0.Alpha19.jar"                           |
==> | "/org/jboss/as/ee/main/wildfly-ee-9.0.0.Alpha2-SNAPSHOT.jar"                                                 |
==> | "/org/jboss/as/ejb3/main/wildfly-ejb3-9.0.0.Alpha2-SNAPSHOT.jar"                                             |
==> | "/org/jboss/as/embedded/main/wildfly-embedded-9.0.0.Alpha2-SNAPSHOT.jar"                                     |
==> | "/org/jboss/as/host-controller/main/wildfly-host-controller-1.0.0.Alpha19.jar"                               |
==> | "/org/jboss/as/jacorb/main/wildfly-jacorb-9.0.0.Alpha2-SNAPSHOT.jar"                                         |
==> | "/org/jboss/as/jaxr/main/wildfly-jaxr-9.0.0.Alpha2-SNAPSHOT.jar"                                             |
==> | "/org/jboss/as/jaxrs/main/wildfly-jaxrs-9.0.0.Alpha2-SNAPSHOT.jar"                                           |
==> | "/org/jboss/as/jdr/main/wildfly-jdr-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/as/jmx/main/wildfly-jmx-1.0.0.Alpha19.jar"                                                       |
==> | "/org/jboss/as/jpa/hibernate/3/jipijapa-hibernate3-1.0.1.Final.jar"                                          |
==> | "/org/jboss/as/jpa/main/wildfly-jpa-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/as/jpa/spi/main/jipijapa-spi-1.0.1.Final.jar"                                                    |
==> | "/org/jboss/as/jsf-injection/main/weld-core-jsf-2.2.9.Final.jar"                                             |
==> | "/org/jboss/as/jsf-injection/main/wildfly-jsf-injection-9.0.0.Alpha2-SNAPSHOT.jar"                           |
==> | "/org/jboss/as/jsf/main/wildfly-jsf-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/as/jsr77/main/wildfly-jsr77-9.0.0.Alpha2-SNAPSHOT.jar"                                           |
==> | "/org/jboss/as/logging/main/wildfly-logging-1.0.0.Alpha19.jar"                                               |
==> | "/org/jboss/as/mail/main/wildfly-mail-9.0.0.Alpha2-SNAPSHOT.jar"                                             |
==> | "/org/jboss/as/management-client-content/main/wildfly-management-client-content-1.0.0.Alpha19.jar"           |
==> | "/org/jboss/as/messaging/main/wildfly-messaging-9.0.0.Alpha2-SNAPSHOT.jar"                                   |
==> | "/org/jboss/as/naming/main/wildfly-naming-9.0.0.Alpha2-SNAPSHOT.jar"                                         |
==> | "/org/jboss/as/network/main/wildfly-network-1.0.0.Alpha19.jar"                                               |
==> | "/org/jboss/as/patching/main/wildfly-patching-1.0.0.Alpha19.jar"                                             |
==> | "/org/jboss/as/platform-mbean/main/wildfly-platform-mbean-1.0.0.Alpha19.jar"                                 |
==> | "/org/jboss/as/pojo/main/wildfly-pojo-9.0.0.Alpha2-SNAPSHOT.jar"                                             |
==> | "/org/jboss/as/process-controller/main/wildfly-process-controller-1.0.0.Alpha19.jar"                         |
==> | "/org/jboss/as/protocol/main/wildfly-protocol-1.0.0.Alpha19.jar"                                             |
==> | "/org/jboss/as/remoting/main/wildfly-remoting-1.0.0.Alpha19.jar"                                             |
==> | "/org/jboss/as/sar/main/wildfly-sar-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/as/security-api/main/wildfly-security-api-9.0.0.Alpha2-SNAPSHOT.jar"                             |
==> | "/org/jboss/as/security/main/wildfly-security-9.0.0.Alpha2-SNAPSHOT.jar"                                     |
==> | "/org/jboss/as/server/main/wildfly-server-1.0.0.Alpha19.jar"                                                 |
==> | "/org/jboss/as/system-jmx/main/wildfly-system-jmx-1.0.0.Alpha19.jar"                                         |
==> | "/org/jboss/as/threads/main/wildfly-threads-1.0.0.Alpha19.jar"                                               |
==> | "/org/jboss/as/transactions/main/wildfly-transactions-9.0.0.Alpha2-SNAPSHOT.jar"                             |
==> | "/org/jboss/as/version/main/jboss-as-version-9.0.0.Alpha2-SNAPSHOT.jar"                                      |
==> | "/org/jboss/as/version/main/wildfly-version-1.0.0.Alpha19.jar"                                               |
==> | "/org/jboss/as/web-common/main/wildfly-web-common-9.0.0.Alpha2-SNAPSHOT.jar"                                 |
==> | "/org/jboss/as/web/main/wildfly-web-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/as/webservices/main/jbossws-cxf-resources-5.0.0.Beta3-wildfly900.jar"                            |
==> | "/org/jboss/as/webservices/main/wildfly-webservices-server-integration-9.0.0.Alpha2-SNAPSHOT.jar"            |
==> | "/org/jboss/as/weld/main/wildfly-weld-9.0.0.Alpha2-SNAPSHOT.jar"                                             |
==> | "/org/jboss/as/xts/main/wildfly-xts-9.0.0.Alpha2-SNAPSHOT.jar"                                               |
==> | "/org/jboss/classfilewriter/main/jboss-classfilewriter-1.0.4.Final.jar"                                      |
==> | "/org/jboss/common-beans/main/jboss-common-beans-1.1.0.Final.jar"                                            |
==> | "/org/jboss/common-core/main/jboss-common-core-2.5.0.Beta1.jar"                                              |
==> | "/org/jboss/dmr/main/jboss-dmr-1.3.0.Beta2.jar"                                                              |
==> | "/org/jboss/ejb-client/main/jboss-ejb-client-2.0.1.Final.jar"                                                |
==> | "/org/jboss/ejb3/main/jboss-ejb3-ext-api-2.1.0.jar"                                                          |
==> | "/org/jboss/genericjms/main/generic-jms-ra-jar-1.0.7.Final.jar"                                              |
==> | "/org/jboss/iiop-client/main/jboss-iiop-client-1.0.0.Final.jar"                                              |
==> | "/org/jboss/integration/ext-content/main/bundled/jboss-seam-int.jar"                                         |
==> | "/org/jboss/invocation/main/jboss-invocation-1.3.0.Final.jar"                                                |
==> | "/org/jboss/ironjacamar/api/main/ironjacamar-common-api-1.2.2.Final.jar"                                     |
==> | "/org/jboss/ironjacamar/api/main/ironjacamar-common-spi-1.2.2.Final.jar"                                     |
==> | "/org/jboss/ironjacamar/api/main/ironjacamar-core-api-1.2.2.Final.jar"                                       |
==> | "/org/jboss/ironjacamar/impl/main/ironjacamar-common-impl-1.2.2.Final.jar"                                   |
==> | "/org/jboss/ironjacamar/impl/main/ironjacamar-core-impl-1.2.2.Final.jar"                                     |
==> | "/org/jboss/ironjacamar/impl/main/ironjacamar-deployers-common-1.2.2.Final.jar"                              |
==> | "/org/jboss/ironjacamar/impl/main/ironjacamar-validator-1.2.2.Final.jar"                                     |
==> | "/org/jboss/ironjacamar/jdbcadapters/main/ironjacamar-jdbc-1.2.2.Final.jar"                                  |
==> | "/org/jboss/jandex/main/jandex-1.2.2.Final.jar"                                                              |
==> | "/org/jboss/jaxbintros/main/jboss-jaxb-intros-1.0.2.GA.jar"                                                  |
==> | "/org/jboss/jboss-transaction-spi/main/jboss-transaction-spi-7.1.0.Final.jar"                                |
==> | "/org/jboss/jts/integration/main/narayana-jts-integration-5.0.4.Final.jar"                                   |
==> | "/org/jboss/jts/main/narayana-jts-idlj-5.0.4.Final.jar"                                                      |
==> | "/org/jboss/log4j/logmanager/main/log4j-jboss-logmanager-1.1.1.Final.jar"                                    |
==> | "/org/jboss/logging/jul-to-slf4j-stub/main/jul-to-slf4j-stub-1.0.1.Final.jar"                                |
==> | "/org/jboss/logging/main/jboss-logging-3.2.1.Final.jar"                                                      |
==> | "/org/jboss/logmanager/main/jboss-logmanager-2.0.0.Beta1.jar"                                                |
==> | "/org/jboss/marshalling/main/jboss-marshalling-1.4.10.Final.jar"                                             |
==> | "/org/jboss/marshalling/river/main/jboss-marshalling-river-1.4.10.Final.jar"                                 |
==> | "/org/jboss/metadata/appclient/main/jboss-metadata-appclient-9.0.0.Beta1.jar"                                |
==> | "/org/jboss/metadata/common/main/jboss-metadata-common-9.0.0.Beta1.jar"                                      |
==> | "/org/jboss/metadata/ear/main/jboss-metadata-ear-9.0.0.Beta1.jar"                                            |
==> | "/org/jboss/metadata/ejb/main/jboss-metadata-ejb-9.0.0.Beta1.jar"                                            |
==> | "/org/jboss/metadata/web/main/jboss-metadata-web-9.0.0.Beta1.jar"                                            |
==> | "/org/jboss/mod_cluster/container/spi/main/mod_cluster-container-spi-1.3.1.Beta1.jar"                        |
==> | "/org/jboss/mod_cluster/core/main/mod_cluster-core-1.3.1.Beta1.jar"                                          |
==> | "/org/jboss/msc/main/jboss-msc-1.2.4.Final.jar"                                                              |
==> | "/org/jboss/narayana/compensations/main/compensations-5.0.4.Final.jar"                                       |
==> | "/org/jboss/narayana/rts/main/restat-api-5.0.4.Final.jar"                                                    |
==> | "/org/jboss/narayana/rts/main/restat-bridge-5.0.4.Final.jar"                                                 |
==> | "/org/jboss/narayana/rts/main/restat-integration-5.0.4.Final.jar"                                            |
==> | "/org/jboss/narayana/rts/main/restat-util-5.0.4.Final.jar"                                                   |
==> | "/org/jboss/remote-naming/main/jboss-remote-naming-2.0.3.Final.jar"                                          |
==> | "/org/jboss/remoting-jmx/main/remoting-jmx-2.0.1.CR1.jar"                                                    |
==> | "/org/jboss/remoting/main/jboss-remoting-4.0.7.Final.jar"                                                    |
==> | "/org/jboss/resteasy/jose-jwt/main/jose-jwt-3.0.10.Final.jar"                                                |
==> | "/org/jboss/resteasy/resteasy-atom-provider/main/resteasy-atom-provider-3.0.10.Final.jar"                    |
==> | "/org/jboss/resteasy/resteasy-cdi/main/resteasy-cdi-3.0.10.Final.jar"                                        |
==> | "/org/jboss/resteasy/resteasy-crypto/main/resteasy-crypto-3.0.10.Final.jar"                                  |
==> | "/org/jboss/resteasy/resteasy-jackson-provider/main/resteasy-jackson-provider-3.0.10.Final.jar"              |
==> | "/org/jboss/resteasy/resteasy-jackson2-provider/main/resteasy-jackson2-provider-3.0.10.Final.jar"            |
==> | "/org/jboss/resteasy/resteasy-jaxb-provider/main/resteasy-jaxb-provider-3.0.10.Final.jar"                    |
==> | "/org/jboss/resteasy/resteasy-jaxrs/main/async-http-servlet-3.0-3.0.10.Final.jar"                            |
==> | "/org/jboss/resteasy/resteasy-jaxrs/main/resteasy-client-3.0.10.Final.jar"                                   |
==> | "/org/jboss/resteasy/resteasy-jaxrs/main/resteasy-jaxrs-3.0.10.Final.jar"                                    |
==> | "/org/jboss/resteasy/resteasy-jettison-provider/main/resteasy-jettison-provider-3.0.10.Final.jar"            |
==> | "/org/jboss/resteasy/resteasy-jsapi/main/resteasy-jsapi-3.0.10.Final.jar"                                    |
==> | "/org/jboss/resteasy/resteasy-json-p-provider/main/resteasy-json-p-provider-3.0.10.Final.jar"                |
==> | "/org/jboss/resteasy/resteasy-multipart-provider/main/resteasy-multipart-provider-3.0.10.Final.jar"          |
==> | "/org/jboss/resteasy/resteasy-spring/main/bundled/resteasy-spring-jar/resteasy-spring-3.0.10.Final.jar"      |
==> | "/org/jboss/resteasy/resteasy-validator-provider-11/main/resteasy-validator-provider-11-3.0.10.Final.jar"    |
==> | "/org/jboss/resteasy/resteasy-yaml-provider/main/resteasy-yaml-provider-3.0.10.Final.jar"                    |
==> | "/org/jboss/sasl/main/jboss-sasl-1.0.5.Final.jar"                                                            |
==> | "/org/jboss/security/negotiation/main/jboss-negotiation-common-2.3.6.Final.jar"                              |
==> | "/org/jboss/security/negotiation/main/jboss-negotiation-extras-2.3.6.Final.jar"                              |
==> | "/org/jboss/security/negotiation/main/jboss-negotiation-net-2.3.6.Final.jar"                                 |
==> | "/org/jboss/security/negotiation/main/jboss-negotiation-ntlm-2.3.6.Final.jar"                                |
==> | "/org/jboss/security/negotiation/main/jboss-negotiation-spnego-2.3.6.Final.jar"                              |
==> | "/org/jboss/security/xacml/main/jbossxacml-2.0.8.Final.jar"                                                  |
==> | "/org/jboss/shrinkwrap/core/main/shrinkwrap-api-1.1.2.jar"                                                   |
==> | "/org/jboss/shrinkwrap/core/main/shrinkwrap-impl-base-1.1.2.jar"                                             |
==> | "/org/jboss/shrinkwrap/core/main/shrinkwrap-spi-1.1.2.jar"                                                   |
==> | "/org/jboss/staxmapper/main/staxmapper-1.1.0.Final.jar"                                                      |
==> | "/org/jboss/stdio/main/jboss-stdio-1.0.2.GA.jar"                                                             |
==> | "/org/jboss/threads/main/jboss-threads-2.2.0.Final.jar"                                                      |
==> | "/org/jboss/vfs/main/jboss-vfs-3.2.9.Final.jar"                                                              |
==> | "/org/jboss/weld/api/main/weld-api-2.2.SP3.jar"                                                              |
==> | "/org/jboss/weld/core/main/weld-core-impl-2.2.9.Final.jar"                                                   |
==> | "/org/jboss/weld/spi/main/weld-spi-2.2.SP3.jar"                                                              |
==> | "/org/jboss/ws/api/main/jbossws-api-1.0.3.CR2.jar"                                                           |
==> | "/org/jboss/ws/common/main/jbossws-common-3.0.0.Beta2.jar"                                                   |
==> | "/org/jboss/ws/cxf/jbossws-cxf-factories/main/jbossws-cxf-factories-5.0.0.Beta3.jar"                         |
==> | "/org/jboss/ws/cxf/jbossws-cxf-server/main/jbossws-cxf-server-5.0.0.Beta3.jar"                               |
==> | "/org/jboss/ws/cxf/jbossws-cxf-transports-udp/main/jbossws-cxf-transports-udp-5.0.0.Beta3.jar"               |
==> | "/org/jboss/ws/cxf/jbossws-cxf-transports-undertow/main/jbossws-cxf-transports-undertow-5.0.0.Beta3.jar"     |
==> | "/org/jboss/ws/jaxws-client/main/jbossws-cxf-client-5.0.0.Beta3.jar"                                         |
==> | "/org/jboss/ws/jaxws-client/main/jbossws-cxf-jaspi-5.0.0.Beta3.jar"                                          |
==> | "/org/jboss/ws/jaxws-undertow-httpspi/main/jaxws-undertow-httpspi-1.0.1.Final.jar"                           |
==> | "/org/jboss/ws/spi/main/jbossws-spi-3.0.0.Beta4.jar"                                                         |
==> | "/org/jboss/ws/tools/common/main/jbossws-common-tools-1.2.1.CR1.jar"                                         |
==> | "/org/jboss/xnio/main/xnio-api-3.3.0.Final.jar"                                                              |
==> | "/org/jboss/xnio/netty/netty-xnio-transport/main/netty-xnio-transport-0.1.1.Final.jar"                       |
==> | "/org/jboss/xnio/nio/main/xnio-nio-3.3.0.Final.jar"                                                          |
==> | "/org/jboss/xts/main/jbosstxbridge-5.0.4.Final.jar"                                                          |
==> | "/org/jboss/xts/main/jbossxts-5.0.4.Final.jar"                                                               |
==> | "/org/jdom/main/jdom-1.1.3.jar"                                                                              |
==> | "/org/jgroups/main/jgroups-3.6.2.Final.jar"                                                                  |
==> | "/org/joda/time/main/joda-time-1.6.2.jar"                                                                    |
==> | "/org/jsoup/main/jsoup-1.7.1.jar"                                                                            |
==> | "/org/kohsuke/rngom/main/rngom-201103.jboss-1.jar"                                                           |
==> | "/org/opensaml/main/opensaml-2.6.1.jar"                                                                      |
==> | "/org/opensaml/main/openws-1.5.1.jar"                                                                        |
==> | "/org/opensaml/main/xmltooling-1.4.1.jar"                                                                    |
==> | "/org/picketbox/main/picketbox-4.9.0.Beta2.jar"                                                              |
==> | "/org/picketbox/main/picketbox-commons-1.0.0.final.jar"                                                      |
==> | "/org/picketbox/main/picketbox-infinispan-4.9.0.Beta2.jar"                                                   |
==> | "/org/picketlink/common/main/picketlink-common-2.7.0.Beta2.jar"                                              |
==> | "/org/picketlink/config/main/picketlink-config-2.7.0.Beta2.jar"                                              |
==> | "/org/picketlink/core/api/main/picketlink-api-2.7.0.Beta2.jar"                                               |
==> | "/org/picketlink/core/main/picketlink-impl-2.7.0.Beta2.jar"                                                  |
==> | "/org/picketlink/federation/bindings/main/picketlink-wildfly8-2.7.0.Beta2.jar"                               |
==> | "/org/picketlink/federation/main/picketlink-federation-2.7.0.Beta2.jar"                                      |
==> | "/org/picketlink/idm/api/main/picketlink-idm-api-2.7.0.Beta2.jar"                                            |
==> | "/org/picketlink/idm/main/picketlink-idm-impl-2.7.0.Beta2.jar"                                               |
==> | "/org/picketlink/idm/schema/main/picketlink-idm-simple-schema-2.7.0.Beta2.jar"                               |
==> | "/org/scannotation/scannotation/main/scannotation-1.0.3.jar"                                                 |
==> | "/org/slf4j/ext/main/slf4j-ext-1.7.7.jbossorg-1.jar"                                                         |
==> | "/org/slf4j/impl/main/slf4j-jboss-logmanager-1.0.3.GA.jar"                                                   |
==> | "/org/slf4j/jcl-over-slf4j/main/jcl-over-slf4j-1.7.7.jbossorg-1.jar"                                         |
==> | "/org/slf4j/main/slf4j-api-1.7.7.jbossorg-1.jar"                                                             |
==> | "/org/wildfly/clustering/api/main/wildfly-clustering-api-9.0.0.Alpha2-SNAPSHOT.jar"                          |
==> | "/org/wildfly/clustering/ee/infinispan/main/wildfly-clustering-ee-infinispan-9.0.0.Alpha2-SNAPSHOT.jar"      |
==> | "/org/wildfly/clustering/ee/spi/main/wildfly-clustering-ee-spi-9.0.0.Alpha2-SNAPSHOT.jar"                    |
==> | "/org/wildfly/clustering/ejb/infinispan/main/wildfly-clustering-ejb-infinispan-9.0.0.Alpha2-SNAPSHOT.jar"    |
==> | "/org/wildfly/clustering/ejb/spi/main/wildfly-clustering-ejb-spi-9.0.0.Alpha2-SNAPSHOT.jar"                  |
==> | "/org/wildfly/clustering/infinispan/spi/main/wildfly-clustering-infinispan-spi-9.0.0.Alpha2-SNAPSHOT.jar"    |
==> | "/org/wildfly/clustering/jgroups/api/main/wildfly-clustering-jgroups-api-9.0.0.Alpha2-SNAPSHOT.jar"          |
==> | "/org/wildfly/clustering/jgroups/spi/main/wildfly-clustering-jgroups-spi-9.0.0.Alpha2-SNAPSHOT.jar"          |
==> | "/org/wildfly/clustering/marshalling/main/wildfly-clustering-marshalling-9.0.0.Alpha2-SNAPSHOT.jar"          |
==> | "/org/wildfly/clustering/server/main/wildfly-clustering-server-9.0.0.Alpha2-SNAPSHOT.jar"                    |
==> | "/org/wildfly/clustering/service/main/wildfly-clustering-service-9.0.0.Alpha2-SNAPSHOT.jar"                  |
==> | "/org/wildfly/clustering/singleton/main/wildfly-clustering-singleton-9.0.0.Alpha2-SNAPSHOT.jar"              |
==> | "/org/wildfly/clustering/spi/main/wildfly-clustering-spi-9.0.0.Alpha2-SNAPSHOT.jar"                          |
==> | "/org/wildfly/clustering/web/api/main/wildfly-clustering-web-api-9.0.0.Alpha2-SNAPSHOT.jar"                  |
==> | "/org/wildfly/clustering/web/infinispan/main/wildfly-clustering-web-infinispan-9.0.0.Alpha2-SNAPSHOT.jar"    |
==> | "/org/wildfly/clustering/web/spi/main/wildfly-clustering-web-spi-9.0.0.Alpha2-SNAPSHOT.jar"                  |
==> | "/org/wildfly/clustering/web/undertow/main/wildfly-clustering-web-undertow-9.0.0.Alpha2-SNAPSHOT.jar"        |
==> | "/org/wildfly/extension/batch/main/wildfly-batch-9.0.0.Alpha2-SNAPSHOT.jar"                                  |
==> | "/org/wildfly/extension/bean-validation/main/wildfly-bean-validation-9.0.0.Alpha2-SNAPSHOT.jar"              |
==> | "/org/wildfly/extension/io/main/wildfly-io-1.0.0.Alpha19.jar"                                                |
==> | "/org/wildfly/extension/mod_cluster/main/wildfly-mod_cluster-extension-9.0.0.Alpha2-SNAPSHOT.jar"            |
==> | "/org/wildfly/extension/picketlink/main/wildfly-picketlink-9.0.0.Alpha2-SNAPSHOT.jar"                        |
==> | "/org/wildfly/extension/request-controller/main/wildfly-request-controller-1.0.0.Alpha19.jar"                |
==> | "/org/wildfly/extension/rts/main/wildfly-rts-9.0.0.Alpha2-SNAPSHOT.jar"                                      |
==> | "/org/wildfly/extension/security/manager/main/wildfly-security-manager-9.0.0.Alpha2-SNAPSHOT.jar"            |
==> | "/org/wildfly/extension/undertow/main/wildfly-undertow-9.0.0.Alpha2-SNAPSHOT.jar"                            |
==> | "/org/wildfly/iiop-openjdk/main/wildfly-iiop-openjdk-9.0.0.Alpha2-SNAPSHOT.jar"                              |
==> | "/org/wildfly/jberet/main/wildfly-jberet-9.0.0.Alpha2-SNAPSHOT.jar"                                          |
==> | "/org/wildfly/mod_cluster/undertow/main/wildfly-mod_cluster-undertow-9.0.0.Alpha2-SNAPSHOT.jar"              |
==> | "/org/wildfly/security/manager/main/wildfly-security-manager-1.1.2.Final.jar"                                |
==> | "/org/yaml/snakeyaml/main/snakeyaml-1.13.jar"                                                                |
==> +--------------------------------------------------------------------------------------------------------------+
==> 404 rows
==> 348 ms
neo4j-sh (?)$
```
