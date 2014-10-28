---
layout: post
title: Hive fails when inserting data to dynamically partitioned table
comments: true
---

Today I spent some time investigating why one of our hive queries failed. Cause: YARN container for map task got killed:
> Container[(...)] is running beyond physical memory limits. Current usage: XXX MB of XXX MB physical memory used; XXX MB of XXX GB virtual memory used. Killing container. (...)

Quick look on configuration which was used to configure dynamic partitioning:

```xml
    <property>
        <name>hive.exec.dynamic.partition.mode</name>
        <value>nonstrict</value>
    </property>
    <property>
        <name>hive.exec.dynamic.partition</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.exec.max.dynamic.partitions</name>
        <value>1000</value>
    </property>
    <property>
        <name>hive.exec.max.dynamic.partitions.pernode</name>
        <value>1000</value>
    </property>
```

First idea: the number of partitions can be limited with a `WHERE` clause. Lets try it out... The number of partitions dropped to ~400 - updated query worked fine. The solution no 1 is to split processing into multiple queries with separate data ranges. Uffff... Lets have another look. There wasn't that much data to insert (~600MB). Hive estimated that it needs only 2 mappers for processing... After some googling I found this [2]. The symptoms are the same - big number of partitions (in my case > 2,5k), container being killed because of high memory usage. I've also made couple interesting observations:

  * `-Xmx` of JVM is set to 1536M - which is way below the container limit (2GB). 
  * The stage which fails is a Map Reduce TableScan stage. And this stage uses only 2 mappers:
    > Hadoop job information for Stage-1: number of mappers: 2; number of reducers: 0

This suggests that this might be a data distribution problem. There is a part of HiveQL which can influence how data distribution is done: `DISTRIBUTE BY` clause. What does it do? [This][3] seems to provide quite nice explanation. After adding `DISTRIBUTE BY` by the dynamic partition Hive stopped failing (hurray!). But what actually happened?

`Hadoop job information for Stage-1: number of mappers: 2; number of reducers: 1`

Turns out the query plan got slightly changed. Execution plan before:

> Stage: Stage-1
>   Map Reduce
>     Alias -> Map Operator Tree:
>       source_table
>         TableScan
>           alias: source_table
>           Filter Operator
>             predicate:
>               expr: (some expr)
>               type: boolean
>             Select Operator
>               expressions:
>                 expr: column_name
>                 type: string
>                 ...
>               outputColumnNames: _col0, ...
>               Select Operator
>                 expressions:
>                   expr: UDFToLong(_col0)
>                   type: bigint
>                   expr: _col1
>                   ...
>                 outputColumnNames: _col0, ...
>               File Output Operator
>               ... (output to file)

Execution plan after:

>  Stage: Stage-1
>    Map Reduce
>      Alias -> Map Operator Tree:
>        source_table 
>          TableScan
>            alias: source_table
>            Filter Operator
>              predicate:
>                expr: (some expression)
>                type: boolean
>              Select Operator
>                expressions:
>                  expr: column_name
>                  type: string
>                  ...
>                outputColumnNames: _col0, ...
>                Reduce Output Operator <- added output operator
>                  sort order: 
>                  Map-reduce partition columns:
>                    expr: _col5 <- column used in DISTRIBUTE BY
>                    type: int
>                  tag: -1
>                  value expressions:
>                    expr: _col0
>                    type: string
>                  ...
>    Reduce Operator Tree: <- added reduce step
>      Extract
>        Select Operator
>          expressions:
>            expr: UDFToLong(_col0)
>            type: bigint
>            ...
>          outputColumnNames: _col0, ...
>          File Output Operator
>          ... (output to file)

Hive added a reduction step after initial scan of table. Processing seems kind of slow (only a single reducer is used - however I'm not sure if this isn't caused by data size). Reducer probably processes partitions separately - what makes it require less memory. One important note: `hive.exec.max.dynamic.partitions` and `hive.exec.max.dynamic.partitions.pernode` still need to be set in such a way that hive doesn't break just because it has too many partitions to process (you need to estimate how many partitions will actually be used).

As a dessert: [Hortonworks documentation][4] suggests turning off memory limits when you hit that kind of problem :)
Short quote:
> Problem: When using the Hive script to create and populate the partitioned table dynamically, the following error is reported in the TaskTracker log file:
> TaskTree [pid=30275,tipID=attempt_201305041854_0350_m_000000_0] is running beyond memory-limits. Current usage : 1619562496bytes. Limit : 1610612736bytes. Killing task. TaskTree [pid=30275,tipID=attempt_201305041854_0350_m_000000_0] is running beyond memory-limits. Current usage : 1619562496bytes. Limit : 1610612736bytes. Killing task. Dump of the process-tree for attempt_201305041854_0350_m_000000_0 : |- PID PPID PGRPID SESSID CMD_NAME USER_MODE_TIME(MILLIS) SYSTEM_TIME(MILLIS) VMEM_USAGE(BYTES) RSSMEM_USAGE(PAGES) FULL_CMD_LINE |- 30275 20786 30275 30275 (java) 2179 476 1619562496 190241 /usr/jdk64/jdk1.6.0_31/jre/bin/java ...
> Workaround: The workaround is disable all the memory settings (...)

[1]: http://www.slideshare.net/ye.mikez/hive-tuning
[2]: https://issues.apache.org/jira/browse/HIVE-6455
[3]: http://stackoverflow.com/questions/18951827/distributed-clause-in-hive
[4]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_5_hive.html
