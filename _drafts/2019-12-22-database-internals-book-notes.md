---
layout: post
title: Notes from Database Internals
tags: [database, storage]
---

## Chapter-1 - Storage Engines - Introduction and Overview

Define the current and anticipated variables such as:

* Schema and record sizes
* Number of clients
* Types of queries and access patterns
* Rates of the read and write queries
* Expected changes in any of these variables

Knowing above variables can help answer the following questions:

* Does the database support required queries?
* Is this database able to handle the amount of data we are planning to store?
* How manu read and write operations can a single node handle?
* How many nodes should the system have?
* How do we expand the cluster given the expected growth rate?
* Whats is the maintenance process?

One of the popular tool for Benchmarking, Performance Evaluation and Comparison is Yahoo Cloud Serving Benchmark (YCSB) https://github.com/brianfrankcooper/YCSB.

* OLTP - Online transaction Processing - Large number of user facing requests and transactions. Queries are often predefined and short lived.
* OLAP - Online Analytical Processing - Complex aggregations. Often used for Analytics, Data warehousing. Queries are often complex ad-hoc and long running.
* HTAP - Hybrid Transactional and Analytical Processing

