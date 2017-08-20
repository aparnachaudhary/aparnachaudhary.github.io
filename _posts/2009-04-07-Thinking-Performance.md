---
layout: post
title: Thinking Performance
tags: [apm, performance-tuning, database]
---

Have you ever spent days rewriting the whole application burning the midnight oil? Well, I don’t think you are the only one. In early days of my career I also made similar mistakes or was a victim of mistakes made by fellow developers. Having taken this roller coaster ride, spending sleepless nights fixing the code, I learned a new mantra – *Tune Early Tune Often*. In most of the enterprise development cycles, performance testing and tuning is done pretty late. Typically we start worrying about it during system testing and by the time (if at all) application goes into UAT phase, performance becomes the critical requirement. So if we are building applications where performance is as important as any other functional requirements, why not invest some time addressing it in early development phase?

Though there are whole bunch of aspects to be considered for performance tuning, in the following post I would talk mainly about the data access.

## Use Data Generator Tools

Make sure every developer workstation has independent database instance running on it. This gives a full control on the test data. Consider using data generators to feed your database with sufficient pseudo-random test data. For non-relational entities we can use http://www.generatedata.com/\#about\[GenerdateData\]. While for generating relational test data, http://dbmonster.kernelpanic.pl/\[DBMonster\] could be the choice.

## O/R Mapper vs JDBC

With the invent of O/R mappers, its quite easy to query data and create a maintainable clean code base. But before using O/R mappers, we should check what kind of features do we need. For instance, hibernate provides support for criteria queries, pagination. But if your application doesn’t need dynamic queries too often, consider using lightweight O/R mappers like iBatis, DButils, or just pure JDBC…ah well Spring JDBC!! Given the performance benefits of JDBC, we might want to avoid O/R mappers for read operations. But for persisting complex parent-child relationships, Hibernate could be the best choice. Try tuning the JDBC Batch / Fetch sizes to minimize the number of database hits.

## Conclusion

In conclusion, make the right choice of data access mechanism. Wherever suitable, use a combination of O/R mappers and JDBC. Make sure you have enough test data to feed the application. Every use case (story in Agile) you develop, make sure it is tested with sufficient test data; so that the performance issues do not accumulate over time. *Tune Early Tune Often!*
