---
layout: post
title: Kafka Confluent Schema Registry
tags: [kafka, schema-registry]
---

Following blog post will cover some of the concepts related to Apache Avro and Confluent Schema Registry.

Why Schema Registry?
--------------------

* Kafka uses bytes and does not perform any data verification; so it is very efficient on CPU
* Kafka does not deal with changes in the data format
* As a side effect; when the data format changes; consumers can no longer process the data
* To avoid such inconsistency issues with data management without impacting the performance and scaling capabilities of Kafka;
Schema Registry component is created by Confluent.

What is Schema Registry?
------------------------
* Schema registry is a component from Confluent
* Supports schema, schema evolution and must be lightweight

Why Avro?
-------------
* JSON supports multiple data formats and is supported by multiple languages
* JSON is less verbose compared to XML; but still can be quite verbose because of limited keys in collections.
* But one limitation of JSON is that there is no schema enforcement
* Avro is a data format to address above limitations of JSON

What is Avro?
-------------
* Avro in simple terms is JSON + Schema
* It is a binary data format
* Data is compressed
* Schema is attached with the data
* Schema's can be documented
* Data can be processed in multiple languages
* Most important aspect supported by Avro is Schema evolution
* Since it is serialized; it is not possible to read the data without a Avro deserializer

Avro Data Types
---------------
* null - no value
* string - unicode char sequence
* boolean - binary value
* int - 32 bit signed integer
* long - 64 bit signed integer
* float - single precision floating point number
* double - double precision floating point number
* bytes - sequence of 8 unsigned bytes
* Enums
  * For modelling fields for which the values will be enumerated
  * Once Enums are defined; changing enum values is forbidden
* Arrays
  * To represent list of undefined size of items
  * Any data type can be used
* Maps
  * List of key values; keys are strings
  * Define type for value
* Unions
  * Unions allow fields to take different types e.g. string, int, etc
  * If defaults are defined; default must be of the type of first item
  * Typical use case is to allow optional values
  * {"name": "salutation", "type": {"null", "string"}, "default": null}

Avro Record Schemas
-------------------

```json
{
     "type": "record",
     "namespace": "io.github.aparnachaudhary",
     "name": "Customer",
     "fields": [
       { "name": "first_name", "type": "string", "doc": "First Name of Customer" },
       { "name": "last_name", "type": "string", "doc": "Last Name of Customer" },
       { "name": "height", "type": "float", "doc": "Height at the time of registration in cm" },
       { "name": "weight", "type": "float", "doc": "Weight at the time of registration in kg" },
       { "name": "automated_email", "type": "boolean", "default": true, "doc": "Set to true if the user is enrolled in marketing emails" }
     ]
}
```

Avro Logical types
------------------
* decimals
* dates -
* time-millis - number of milliseconds from midnight
* timestamp-millis - number of milliseconds from epoch time
* { "name": "registered_on", "type": "long", "logicalType":"timestamp-millis", "doc": "Time of registration in milliseconds since epoch time" },


Schema Evolution types and guidelines
-------------------------------------

Schema Evolution Types:

* Backward - Read old data with new Schema
* Forward - Read new data with Old schema
* Full - both backward and forward compatible
* None/Breaking - No compatibility


General Guidelines:

* Make primary key a required field
* Use default values for fields that could be removed in future
* Never ever change enum values
* Never rename fields; use aliases instead
* When adding new fields; always use default values
* Never delete a required field