---
layout: post
title: Cassandra - first touch
comments: true
---

### What is Cassandra?

* NoSQL data-storage engine
* Stores information as key-value pairs
* Has masterless architecture (no masters no slaves)
* Analog of Amazon Dynamo DB or Google's BigTable
* Based on the famous Dynamo paper from Amazon and famous BigTable paper from
  Google (actually blends them)

### Architecture

* All nodes participate in a cluster
* No single point of failure, shared nothing 
* Nodes are organized as a ring
* Scales linearly (add more nodes == add more capacity or make the cluster
  faster)
* Does asynchronous replication
* Can work in muliple data centers in active-active mode
* Uses _consistent hashing_ to spread data over the cluster.
* Data is replicated N times - N is the _replication factor_.
* How replication works?
* The client can set the consistency level (ONE, QUORUM, LOCAL_QUORUM,
  LOCAL_ONE, TWO, ALL)
* Data can be accessed/inserted with Cassandra Query Language which looks a lot
  like SQL.
* Cassandra is an OLTP system.
  * INSERT always does an overwrite (it doesn't check if row of data exists
    already - just writes); however this can be forced with some other syntax.

### What does a single node do?

* Write: 
  * first the data is dumped to commit log (append only - super fast)
  * then it updates the table in memory
  * this triggers an ACK sent to client
  * then the data is dumped to a file on a disk (again those are append only
    big files)
* After dumping data to harddrive happened over a period of time, it is time
  for compaction (merge sort over files on disk, then we choose the records
  with the most recent timestamps and voila)
* To which partition a row goes? Cassandra does an MD5 on the PK which gives a
  128 bit number. Every node is responsible for handling a subset of the key
  pace. Because we always do the MD5 on primary key we always end up on the
  same node -> we have _consistent hashing_.

### When to use it?

* Collecting enormous amount of data
  * The amount of writes handled by the cluster scales linearly with the number
    of nodes in the cluster.
* When a single box is not enough anymore
* ... and sharding is not good enough for scaling

### Differences to HBase

* A lot of similarities - they ultimately originate from BigTable paper.
* Cassandra is easier to run (operationally - because all nodes are created
  equal)
* HBase has advantages when you are doing operations which are sequential
* Programmatically - CQL seems more expressive
* MapReduce - there is some integration for Cassandra, HBase is a native
  because storage is in HDFS

### Links

* [wikipedia](https://en.wikipedia.org/wiki/Apache_Cassandra)
* [official page](http://cassandra.apache.org)
* [good intro talk](https://www.youtube.com/watch?v=B_HTdrTgGNs)
* [Dynamo
  paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
