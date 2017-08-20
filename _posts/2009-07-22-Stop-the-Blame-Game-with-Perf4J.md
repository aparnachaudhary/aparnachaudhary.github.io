---
layout: post
title: Stop the Blame Game with Perf4J
tags: [logging, performance]
---

Performance tracking and monitoring is often a problem with many applications. Especially in systems with distributed SOA architecture, its difficult to identify the services causing performance hit. So its nice, wise and safe always to profile heavy operations in exterprise applications. Theres always a bad day when suddenly system starts showing its true colors by giving sub-optimal performance in production environment. Life is easy at this point, if all the components have runtime performance metrics available for analysis.

Typically, the performance metrics are added in code using System.currentTimeMillis().


```java
long startTime = System.currentTimeMillis(); // execute the block of code to be timed log.info("Performanceic for XYZ Block: " + (System.currentTimeMillis() - startTime));
```

And then custom code is developed, to produce the aggregated statistics like avg, min, max, and number of invocations. It doesn’t stop here. You really cannot present statistics in this crude format to senior management. So some more time and efforts are spent on getting the right charting tool.

http://perf4j.codehaus.org/index.html\[Perf4J\] is the answer to above requirements. It ships with a simple StopWatch mechanism to capture timings. The command line tool parses log files and generates aggregated stats. The usage is quite simple.

```java
java -jar perf4j-0.9.10.jar times.log
```

The command line tool also supports generating the CSV file.

```java
java -jar perf4j-0.9.10.jar -f csv times.log
```

And generating the graphs is so damn easy. The Mean and TPS timing charts are generated with command line parser. Though the possible graph types are Mean, Min, Max, StdDev, Count and TPS.

```java
java -jar perf4j-0.9.10.jar --graph perfGraphs.out times.log
```

Another nice feature of Perf4J is the ability to expose performance statistics as JMX attributes. Its also possible to set thresholds on these attributes and then send notifications when statistics exceed specified thresholds.

In conclusion, with Perf4J you can get rid of all the boilerplate code written to generate performance statistics. Currently it supports only log4j appenders but in future releases, we can expect more appenders and handlers. Perf4J sounds a promising tool for performance analysis in production environment.

Happy Instrumentation!!
