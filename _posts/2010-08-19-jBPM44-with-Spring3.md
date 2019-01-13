---
layout: post
title: jBPM4.4 with Spring3
tags: [enterprise-integration, jbpm, jbpm]
---

I’m working on a jBPM prototype since last couple of days. The integration with Spring wasn’t a smooth ride. In my previous projects, I used jBPM3.2. But it seems jBPM4 is pretty much a rewrite. There are couple of few nice additions like service API’ for task and process management, support for java task and many more. And there are few changes which I didn’t like. Till 4.2 the support for JPA was provided, which is taken off starting 4.3 release. Yes definitely you can write some wrappers and still achieve it. But!! Between 4.3 and 4.4 also there are some major API changes. So the whole point is to get the app running, I had to digg into mailing lists and source code. Now that it works, I thought I would write a short blog about it.

## Sample Workflow

To get started with this setup, I created a very basic defect tracking workflow as shown in the diagram below.

![](/img/jbpm4-spring.png)

## Maven Dependencies

Add the following jBPM specific dependencies.

```xml
<!-- JBPM DEPENDENCIES -->
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>jsr250-api</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.jbpm.jbpm4</groupId>
    <artifactId>jbpm-test-base</artifactId>
    <version>${jbpm.version}</version>
</dependency>
<dependency>
    <groupId>org.jbpm.jbpm4</groupId>
    <artifactId>jbpm-jpdl</artifactId>
    <version>${jbpm.version}</version>
</dependency>
<dependency>
    <groupId>org.jbpm.jbpm4</groupId>
    <artifactId>jbpm-pvm</artifactId>
    <version>${jbpm.version}</version>
</dependency>
```

## jBPM Configuration

Couple of files are required for configuring jBPM.

*jbpm.cfg.xml*

```xml

<?xml version="1.0" encoding="UTF-8"?>
<jbpm-configuration>

    <!-- jBPM Configuration Files -->
    <import resource="jbpm.default.cfg.xml" />
    <import resource="jbpm.businesscalendar.cfg.xml" />
    <import resource="jbpm.tx.hibernate.cfg.xml" />
    <import resource="jbpm.jpdl.cfg.xml" />
    <import resource="jbpm.identity.cfg.xml" />
 
    <!-- Spring Specific Configuration -->
    <process-engine-context>
        <command-service name="newTxRequiredCommandService">
            <retry-interceptor />
            <environment-interceptor policy="requiresNew" />
            <spring-transaction-interceptor
                policy="requiresNew" />
        </command-service>
 
        <command-service name="txRequiredCommandService">
            <retry-interceptor />
            <environment-interceptor />
            <spring-transaction-interceptor />
        </command-service>
    </process-engine-context>
 
    <transaction-context>
        <transaction type="spring" />
        <hibernate-session current="true" />
    </transaction-context>
 
</jbpm-configuration>
```

*jbpm.hibernate.cfg.xml*

Currently I’ve specified all the connection details in this file (ugly). At this moment I’m not yet sure how to externalize these details. There must be a way. Feel free to drop a comment if you know how to do that.

```xml

<?xml version="1.0" encoding="utf-8"?>
 
<!DOCTYPE hibernate-configuration PUBLIC
          "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
          "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
 
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQLDialect</property>
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.connection.url">jdbc:postgresql://localhost/jbpm-demo</property>
        <property name="hibernate.connection.username">postgres</property>
        <property name="hibernate.connection.password">postgres</property>
        <property name="hibernate.format_sql">true</property>
 
        <mapping resource="jbpm.repository.hbm.xml" />
        <mapping resource="jbpm.execution.hbm.xml" />
        <mapping resource="jbpm.history.hbm.xml" />
        <mapping resource="jbpm.task.hbm.xml" />
        <mapping resource="jbpm.identity.hbm.xml" />
    </session-factory>
</hibernate-configuration>
```

## Wiring it up with Spring

jbpm4-context.xml

This file contains the spring specific configuration for jBPM. This file can be used as is for any jBPM4.4 project.

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
 
    <!-- ========================================================= -->
    <!-- =================== jBPM Bean Definitions =============== -->
    <!-- ========================================================= -->
    <bean id="springHelper" class="org.jbpm.pvm.internal.processengine.SpringHelper">
        <property name="jbpmCfg">
            <value>jbpm.cfg.xml</value>
        </property>
    </bean>
 
    <bean id="processEngine" factory-bean="springHelper"
        factory-method="createProcessEngine" />
 
    <bean id="repositoryService" factory-bean="processEngine"
        factory-method="getRepositoryService" />
 
    <bean id="executionService" factory-bean="processEngine"
        factory-method="getExecutionService" />
 
    <bean id="taskService" factory-bean="processEngine"
        factory-method="getTaskService" />
 
    <bean id="historyService" factory-bean="processEngine"
        factory-method="getHistoryService" />
 
    <bean id="managementService" factory-bean="processEngine"
        factory-method="getManagementService" />
 
</beans>
```

Once you have these configurations in place, you just need to import the jbpm4-context.xml in your spring application context and configure the Hibernate session factory and transaction managers, etc.

## Using Process Definition in Services

```java

// removed other dependencies and imports
public class DefectServiceImpl implements DefectService {
 
    private static final String DEFECT_TRACKING_PROCESS_KEY = "DefectTracking";
    private List<String> processDefinitions;
 
    @Autowired
    private RepositoryService repositoryService;
    @Autowired
    private ExecutionService executionService;
    @Autowired
    private TaskService taskService;
 
    @PostConstruct
    public void setupProcessDefinitions() {
        try {
            for (String processDefinition : processDefinitions) {
                NewDeployment deployment = repositoryService.createDeployment();
                deployment.addResourceFromUrl(new ClassPathResource(processDefinition).getURL());
                deployment.deploy();
            }
        } catch (IOException e) {
            logger.info("IOException occurred: ", e);
            throw new RuntimeException("An error occured while trying to deploy a process definition", e);
        }
    }
 
    @Override
    @Transactional(readOnly = false)
    public Defect createDefect(String description, String createdBy, String assignedTo) {
        Defect defect = new Defect();
        defect.setDescription(description);
        defect.setAssignedTo(assignedTo);
        defect.setCreatedBy(createdBy);
        defect.setCreatedDate(new LocalDate());
        defect.setStatus(DefectStatus.NEW);
        defectRepository.save(defect);
 
        Map<String, Object> vars = new HashMap<String, Object>();
        vars.put("defectId", defect.getId());
        ProcessInstance processInstance = executionService.startProcessInstanceByKey(DEFECT_TRACKING_PROCESS_KEY, vars,
                Long.toString(defect.getId()));
        logger.debug("Task assigned to manager1: " + taskService.findPersonalTasks("manager1"));
        return defect;
    }
 
    /**
     * A simple mutator to facilitate configuration.
     *
     * @param definitions
     */
    public void setProcessDefinitions(List<String> definitions) {
        this.processDefinitions = definitions;
    }
```

## Service declaration in application context

```xml

<bean id="defectService" class="com.tenxperts.demo.service.impl.DefectServiceImpl">
    <property name="processDefinitions">
        <list>
            <value>/process/DefectTracking.jpdl.xml</value>
        </list>
    </property>
</bean>
```

With these configurations in place, now you are all set to use jBPM easily in your spring services. The jBPM installation has some nice examples which you can use. But again it misses some common examples like using application user management instead of relying on jBPM identity service, using spring services directly in jPDL, etc. I would try to share some samples for these use cases in my next post.

Feel free to drop a comment if you need any assistance with the configurations or if you have any suggestions to improve the above sample.

## References

* http://www.amazon.com/Spring-Enterprise-Recipes-Problem-Solution-Approach/dp/1430224975[Spring Enterprise Recipes]
* http://diversit.eu/2010/01/10/jbpm-4-3-with-spring/[jBPM4.3 with Spring]

## Source Code

You can get the source for this article on my github repository https://github.com/aparnachaudhary/jbpm-spring-demo.


