---
layout: post
title: Book Notes Architecting HBase Applications
tags: [book, HBase]
---

Recently I read the book "Architecting HBase Applications". I tried to capture the notes in the following post. The book is a good read and I would recommend you to buy one. http://shop.oreilly.com/product/0636920035688.do

## Chapter 1 - What is HBase?

* HBase is built to be a fault-tolerant application hosting large tables of sparse data in the order of trillions of rows and millions of columns. 
* It allows low latency and near real time random reads and writes.
* HBase is designed with Availability over Consistency.
* HBase provides atomic and strongly consistent row level operations.
* It is not a traditional transactional database.
* HBase supports dynamic schema in the sense that while creating a table only table name and column families need to be defined. Columns can be allocated dynamically and can vary per row.

## Chapter 2 - HBase Principles

* 
