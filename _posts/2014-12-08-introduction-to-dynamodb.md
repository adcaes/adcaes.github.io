---
layout: post
title: "Getting started with DynamoDB"
date:   2014-12-08
tags: [AWS]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
  show_in_list: false
---

DynamoDB is a managed NoSQL database offered by Amazon as part of AWS. It is basically a key-value store with some nice additions, the most important, the ability to automatically maintain indexes over different attributes.

It is fast, scalable, and relatively cheap. The pricing model is based on throughput and storage. Amazon's documentation is a bit complex, so here is a simplified description of DynamoDB that should be enough to get started.

## Data Model

DynamoDB data model is based on schema-less tables that store rows and can be queried using indexes.

A table in DynamoDB needs a primary key, and the primary key always enforces a unique constraint. A primary key consists of one or two attributes:

* A mandatory field, the hash key.
* An optional field, the range key.

Apart from the primary key, a table can have additional indexes, called secondary indexes. There are two types of secondary indexes:

* Local secondary index: an index that has the same hash key as the primary key of the table, but a different range key.
* Global secondary index: an index with a hash and range key that can be different from those of the primary key of the table. Global secondary indexes can be seen as new tables that are automatically synced with the original table. When creating a global secondary index, it has to be specified whether to synch the entire table, or just the fields that are part of the primary key, or manually pick which fields to copy. In general, if your rows are smaller than 1KB, just sync all the data, as it is going to cost you almost the same (more on this in Pricing).
Note that opposed to the primary key, global secondary indexes don't impose a unique constraint. It is also important to take into account a big limitation of DynamoDB, indexes have to be defined when the table is create and it is not possible to add or delete an index to an existing table.

## Query and Scan

There are two basic operations for reading data from DynamoDB, query and scan.

### Query

A query operation gets items from a table using one of its indexes. A query operation must specify the index to use, the value of the hash key, and optionally the value of the range key with one of the available filtering operators (EQ | LE | LT | GE | GT | BEGINS_WITH | BETWEEN).

You read it right, **the hash key always has to be provided, and it is evaluated with an equality condition**. So if you need to perform complex queries, like getting all the rows created in the last hour, DynamoDB is not a good option.

You may wonder if you can walk around this limitation by defining an index with an extremely generic hash key, for example a constant, and use the filtering operators offered by the range key to perform more complex queries.

The problem is that is not a good idea to have a significant amount of rows mapped to the same hash key value. The hash key is used to distribute the entries of a table among different nodes, so if many entries are mapped to the same hash key, you lose the scalability and speed offered by DynamoDB.

So keep in mind that DynamoDB query capabilities are limited: the hash key is always filtered with an equality condition and even though the range key can be filtered with different operators, it has to be used together with the hash key.

### Scan

A scan operation examines every item in the table. A single scan request can retrieve a maximum of 1 MB of data, to retrieve more data you need to do multiple requests with pagination.

### Consistency

By default, read operations (query and scan) are eventually consistent. This means that the returned values may not reflect a recently completed write. According to the documentation, consistency across all copies of data is usually reached within a second.

In contrast, a strongly consistent read returns a result that reflects all writes that received a successful response prior to the read.

When performing a read operation, it is possible to specify that it has to be strongly consistent, though strongly consistent reads are not supported on global secondary indexes.

## Pricing

The pricing model of DynamoDB is a bit peculiar, you are charged per table for three different concepts:

* Storage
* Read capacity
* Write capacity

Storage cost is straightforward, being billed per GB, and it is is extremely cheap, currently 0.25$/GB per month, having 25 GB in the free tier.*

Capacity pricing is more complex. You pay for the provisioned (aka. maximum) throughput of the table, not the used capacity. The good think though is that you can change the capacity of the table as frequently as you want, and it only takes a few minutes for the new capacity to be available, with zero downtime.

Read capacity is the number of strongly consistent reads/second of items of size <= 4 KB. Eventually consistent reads consume only 0.5 read capacity units. If your rows are smaller than 4 KB, when performing a strongly consistent read, each fetched row will consume one read unit.

Write capacity is the number of writes/second of items of size <= 1 KB. When writing into the table each secondary index consumes an additional write unit. So if a table has one local secondary index and one global secondary index, adding or updating a row of size <= 1KB will consume 3 write units, 1 for the table and 2 for the indexes.

Capacity is billed hourly and the price is:

* $0.0065 per hour for every 10 units of write capacity ($0.00065 write unit/hour).*
* $0.0065 per hour for every 50 units of read capacity ($0.00013 read unit/hour).*

For example, if your table needs to support 10 writes/s and 100 strongly consistent reads/s, and the items stored are <= 1KB, it would cost:

10 write units x $0.00065 + 100 read units x $0.00013 = $0.0195/hour or $14.04/month.

*\*These prices are for US East region. To see the current prices for a specific AWS region check out this page.*
