---
layout: post
title: Using mini-clusters to do integration testing in Hadoop
comments: true
---

One of the most painful things for me, while working with Apache Hadoop, is testing. Until recently, to test eg. a Hive query or a MR job I had to use a VM with all things installed inside. Thankfully, this has changed when I found [this][1]. Turns out you can create nice in-memory mini clusters with HBase/HDFS/YARN (at least this is the part I was playing around). I created [this][2] sample project to document how to make it working (gradle build definition included!). It contains only a single test case which adds some rows to a test HBase table and then checks if the table actually contains them.

Code is based on CDH 4.7.0, however it should work if You just bump up versions or switch to artifacts from other providers.

[1]: http://hbase.apache.org/book/ch19s04.html
[2]: https://github.com/pkoperek/hadoop-minicluster-sample
