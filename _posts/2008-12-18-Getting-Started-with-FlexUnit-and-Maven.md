---
layout: post
title: Getting Started with FlexUnit and Maven
tags: [flex, mate, maven]
---

Today I created my first unit tests for Flex code using FlexUnit. Later I integrated the tests with the maven build using Flex Mojos. Flex-Mojos is a collection of maven-plugins created to work with Flex.

We have to use the Flex Compiler Plugin. This plugin is basically used to compile the source files and run tests. This plugin has 5 goals which are bound to different maven lifecycles. I chose to explicitly specify goals that I want maven to run. Maven has a default value for testSourceDirectory as src/test/java. For using Flex mojos, we have to specify testSourceDirectory as src/test/flex. Make sure all your tests are created under this directory.

```xml
<build> 
<sourceDirectory>src/main/flex</sourceDirectory> 
<testSourceDirectory>src/test/flex</testSourceDirectory> 
<plugins> 
<plugin> 
<groupId>info.flex-mojos</groupId> 
<artifactId>flex-compiler-mojo</artifactId> 
<version>${flex-mojos.version}</version> 
<extensions>true</extensions> 
<executions> 
<execution> 
<goals> 
<goal>compile-swf</goal> 
<goal>test-compile</goal> 
<goal>test-run</goal> </goals> 
</execution> 
</executions> 
<configuration> 
<locales> 
<param>en\_US</param> 
</locales> 
<contextRoot>/</contextRoot> 
</configuration> 
</plugin> 
</plugins> 
</build> 
```

Next thing is to add Flex Unit dependency in the dependency section.

```xml
    <dependencies>
        <dependency>
            <groupId>flexunit</groupId>
            <artifactId>flexunit</artifactId>
            <version>0.85</version>
            <type>swc</type>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>flexunit.junit</groupId>
            <artifactId>flexunit-optional</artifactId>
            <version>0.85</version>
            <type>swc</type>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

You would also need flash player installed on the build server. In case if you end up with OutOfMemory error, increase the heap size using MAVEN\_OPTS in mvn.bat file.

```bash
SET MAVEN\_OPTS=-Xmx128m
```

If you need more help, you can always refer the http://svn.sonatype.org/flexmojos/trunk/rvin-mojo/test-harness/projects/concept/flexunit-example/\[FlexUnit Example\].
