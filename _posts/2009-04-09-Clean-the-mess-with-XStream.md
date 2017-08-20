---
layout: post
title: Clean the mess with XStream
tags: [unit-testing]
---

Writing clean, isolated and efficient unit test is often a challenge for developers. Efficient test should cover all the possible business scenarios. To create test for covering multiple test scenarios, you need more test data.

For instance, imagine you are writing a test for some Service component. Now if the responsibility of this service is to just collect some data from DAO Layer and pass it on to Business Delegate, life is easy. You can create a mock for DAO using frameworks like http://easymock.org/\[EasyMock\] and you are done. But that’s not often the case. For testing services with complex business logic, its not sufficient to return dummy data. In this case, we create the expected test data and mock the DAO to return the expected data. If this input seed is a simple object, its few additional lines of code and we are done. But what if the test is dealing with complex data model? Normal practice that I observed amongst developers is some private methods are created deep down the test to generate test data – generateXXX().

Situation becomes more worse when after few months there are some changes in the domain model. The developer opens the test and the thought to fix the clumsy code can really make him fussy. If he is not religious about unit testing, he might try to fix the test data instead of test and the whole motivation is at stake. Don’t worry. XStream @ Rescue.

http://xstream.codehaus.org/index.html\[XStream\] is a simple library to serialize objects to XML and back again. It supports de-serialization to nested object structure.

Input XML:

```xml
<person> <firstname>Joe</firstname> <lastname>Walnes</lastname> <phone> <code>123</code> <number>1234-456</number> </phone> <fax> <code>123</code> <number>9999-999</number> </fax> </person>
```

Conversion Code:

```java
XStream xstream = new XStream(); xstream.alias("personPePerson.class); xstream.alias("phonePhPhoneNumber.class); Person newJoe = (Person)xstream.fromXML(xml);
```

As you can see, creation of nested objects is extremely easy with XStream. We just need to create aliases for Custom classes to XML elements. That’s the only configuration required to work with XStream. XStream is lightweight, simple and does not support validations. If you need XML to Object conversion with validations and complex business rules, http://commons.apache.org/digester/\[Commons Digester\] could be the choice.

With this approach you can create a XML for the test scenario and de-serialize in the test. The test would definitely take a bit longer to execute but is more clean, readable and maintainable.
