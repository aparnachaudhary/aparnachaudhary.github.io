---
layout: post
title: Book Notes Architecting HBase Applications
tags: [book, HBase]
---

Recently I read the book "Architecting HBase Applications". I tried to capture the notes in the following post. The book is a good read and I would recommend you to buy one. http://shop.oreilly.com/product/0636920035688.do

## Chapter 1 - What is HBase?

* HBase is built to be a fault-tolerant application hosting large tables of sparse data in the order of trillions of rows and millions of columns. 
* It allows low latency and near real time random reads and writes.
* HBase provides atomic and strongly consistent row level operations.
* It is not a traditional transactional database.
* HBase is a column oriented database.
* HBase supports dynamic schema in the sense that while creating a table only table name and column families need to be defined. Columns can be allocated dynamically and can vary per row.

## Chapter 2 - HBase Principles

* HBase stores data in tables and has a concept of row key and column names.
* There are two types of tables - system and user.
  * system tables are used for storing meta information like ACL, metadata for the tables, regions, namespaces, etc
  * user tables are used for storing data
* HBase table consists of one or more column families.
* Columns with no data are not stored with null values like traditional databases, instead they are not stored on the disk. This saves storage space while dealing with sparse datasets.
* A combination of column and row key is called as a Cell.
* To allow faster access, keys and columns are alphabetically sorted within a table.
* Keys are sorted based on byte values.
* Since table names and column family names are used to create files and directories, they must use printable characters. Same is recommended for column names though not mandatory.
* Storage layout: 
  * Table --> One to Many Regions 
  * Region --> One to many Column Families. 
  * Column family --> single store
  * Store --> unique memstore and one or many HFiles
* So on a single region server, there will be multiple memstores based on number of column families
* To achieve scalability, tables are split/sharded into multiple regions.
* Regions are assigned to a region server.
* Each region has a start and end row key.
* This information is stored in hbase:meta table.
* Regions are balanced (split/merged) based on their size.
* Data related to each column family is stored in a separate file.
* It is recommended to group the data with the same format and same access pattern in the same column family.
  * For example, textual data can be stored in one column family with compression enabled.
  * while binary data like images could be stored in separate column family
* Typically it is recommended to rethink about schema's with more than 2 column families. Many production systems do not need more than 2 column families. Probably splitting the table might be better in such situations. Unnecessary use of column families can lead to small files.
* Data is first written to memstore. Once memstore is full and must be flushed to disk, HFile is created.
* HFiles are then further compacted to bigger files.
* HFiles are stored in HDFS.
* HFiles consists of multiple blocks. They are not the same as HDFS blocks. Each HFile block can be between 8KB and 1MB - default size is 64KB.
* Based on the compression configuration, size of data at rest might vary. Irrespective of compression configuration, block size is always 64KB.
* Larger blocks create smaller number of index valyes and are good for sequential reads.
* Smaller blocks are good for random read access.
* 

