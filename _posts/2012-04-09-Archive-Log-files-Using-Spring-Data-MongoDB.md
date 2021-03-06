---
layout: post
title: Archive Log files Using Spring Data MongoDB
tags: [mongodb, gridfs, unstructured-data, database]
---

MongoDB was on my ToDo list for a while. Finally I decided to stop procrastinating and give it a spin. Since simple hello-world app is no fun, I was thinking of some real life use cases of NoSQL database. One simple use case which I could think of was to create a centralised log store. MongoDB already has a Log4j appender using which application logs can be directly written to MongoDB. But archiving the existing logs in MongoDB is something which is not available out of the box. Also sometimes it’s convenient to write application logs to regular log files but use a store like MongoDB for archival purpose.

Being a big fan of Spring Framework, I decided to use SpringData MongoDB module. Now I don’t have to worry about writing all the boilerplate code to deal with MongoDB. Also I use Spring Integration for file polling mechanism. The following blog post elaborates how I use spring-data-mongodb along with spring-integration framework to archive log4j log files in a mongodb store.

## Overview

Following picture explains a high level set-up of the prototype. I use File Inbound Channel Adapter to read the log files. When a new file is available on the fileChannel, a service activator component, in this case LogProcessor is invoked. It parses the log file and creates LogRecord objects. These LogRecord objects are stored in MongoDB using LogRecordRepository. Once the parsing and storage is done, the file is moved to a processed folder using File Outbound Channel Adapter. This would prevent duplicate processing of the log files.

![](/img/logarchiverusingmongodb.jpg)

## Set Up MongoDB

Before we start with the code, let’s first setup MongoDB. MongoDB can be started using following command.

```bash

$MONGO_HOME/bin/mongod --dbpath /Developer/Softwares/mongodb/data/db/
```

MongoDB also provides a simple REST based admin interface. This can be enabled by starting mongodb with following command.

```bash

$MONGO_HOME/bin/mongod --dbpath /Developer/Softwares/mongodb/data/db/ --rest
```

MongoDB admin console can be accessed using http://localhost:28017. Or the content of collection can be viewed using http://127.0.0.1:28017/databaseName/collectionName/.

To view the data in MongoDB, couple of client applications are available online. I personally use MongoExplorer.

## Running the demo

The source code for the prototype is available on github log-archiver. Run the prototype using mvn exec:java. Login to MongoExplorer. You can see the log entries are available in ‘test’ database under ‘logRecord’ collection.


![](/img/mongoexplorer.png)


## Understanding the code

Spring Integration Configurations:

```xml

<!-- File Poller component -->
<file:inbound-channel-adapter id="filesIn"
    directory="file:data" prevent-duplicates="true" auto-startup="true"
    channel="fileChannel" filename-pattern="*.log">
    <integration:poller fixed-rate="10"
        max-messages-per-poll="1" />
</file:inbound-channel-adapter>
 
<!-- Service activator to process the file -->
<integration:service-activator id="logParser"
    input-channel="fileChannel" output-channel="moveFile" ref="logProcessor">
</integration:service-activator>
 
<!-- Moves the file to different folder to prevent from duplicate processing -->
<file:outbound-channel-adapter id="moveFile"
    directory="file:data/processed" delete-source-files="true" />
```

MongoDB Configurations:

```xml

<!-- ===================================================== -->
<!-- MONGODB SETUP -->
<!-- ===================================================== -->
<bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
    <constructor-arg name="mongo" ref="mongo" />
    <constructor-arg name="databaseName" value="test" />
</bean>
 
<!-- Factory bean that creates the Mongo instance -->
<bean id="mongo" class="org.springframework.data.mongodb.core.MongoFactoryBean">
    <property name="host" value="127.0.0.1" />
    <property name="port" value="27017" />
</bean>
 
<!-- Use this post processor to translate any MongoExceptions thrown in
    @Repository annotated classes -->
<bean
    class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor" />

```


## MongoDB Repository

Creating a Document object is quite simple. Just annotate the java bean with @Document and the id field as @Id. This field maps to “_id” field in Mongo document. If the id field is not set, the driver will assign a ObjectId with a generated value.

```java

@Document
public class LogRecord {
 
    @Id
    private String id;
```

For performing CRUD operations, we can define a repository interface. The Repository should extend CrudRepository provided by the framework.

```java

public interface LogRecordRepository extends CrudRepository<LogRecord, String> {
 
    /**
     * Creates a new collection for LogRecord entity.
     */
    void createCollection();
 
    /**
     * Drops LogRecord collection.
     */
    void dropCollection();
}
```


MongoTemplate provides a simple way for you to save, update, and delete domain objects and map those objects to documents stored in MongoDB.

```java
@Repository("logRecordRepository")
public class LogRecordRepositoryImpl implements LogRecordRepository {
 
    @Autowired
    MongoOperations mongoOperations;
 
    public void createCollection() {
        try {
            if (!mongoOperations.collectionExists(LogRecord.class)) {
                mongoOperations.createCollection(LogRecord.class);
            }
        } catch (Exception e) {
            logger.error("Failed to create mongo-collection", e);
            throw new RuntimeException("Failed to create mongo-collection", e);
        }
    }
```

## Conclusion

In the above blog post I demonstrated how to setup MongoDB and use spring-integration file component along with spring-data-mongodb to archive application log files. You can see that all the boilerplate code is handled by spring-framework. The configurations required to setup spring-data-mongodb module are quite simple. Also with few annotations, it’s quite easy to convert java beans to Document objects.

Next on my cards is to experiment with the indexing and search feature of MongoDB.


## References

* http://www.springsource.org/spring-data/mongodb[Spring Data MongoDB documentation]
* http://openmymind.net/2011/3/28/The-Little-MongoDB-Book[The Little MongoDB book]
* http://code.google.com/p/otroslogviewer[Otros Log Parsing Framework]
