Problem: YARN map container gets killed (it is one of hive query stages):
Dynamic partition config:
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
    <value>2200</value>
    <description>Maximum number of dynamic partitions allowed to be created in total.</description>
    </property>
    <property>
    <name>hive.exec.max.dynamic.partitions.pernode</name>
    <value>2200</value>
    <description>Maximum number of dynamic partitions allowed to be created in each mapper/reducer node.</description>
    </property>

if there are ~400 partitions - works fine (tuned with WHERE clause in query). Probably this is that problem: [2]. The symptoms are the same - big number of partitions (> 2,5k), container being killed because of high mem usage. The interesting thing is that -Xmx is set to 1536M - which is way below the container limit (2GB). Moreover: the stage which fails is a Map Reduce TableScan stage. And this stage uses only 2 mappers:

> Hadoop job information for Stage-1: number of mappers: 2; number of reducers: 0

The problem can be also solved with the DISTRIBUTE BY clause. Hive will add a reducer then:

> Hadoop job information for Stage-1: number of mappers: 2; number of reducers: 1

Execution plan before:

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

To make it working You need to remember to set hive.exec.max.dynamic.partitions/.pernode in such a way that hive doesn't crap out with an error.

Dessert: Hortonworks suggests turning off memory limits when you hit this problem :)

http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_5_hive.html

> Problem: When using the Hive script to create and populate the partitioned table dynamically, the following error is reported in the TaskTracker log file:
> TaskTree [pid=30275,tipID=attempt_201305041854_0350_m_000000_0] is running beyond memory-limits. Current usage : 1619562496bytes. Limit : 1610612736bytes. Killing task. TaskTree [pid=30275,tipID=attempt_201305041854_0350_m_000000_0] is running beyond memory-limits. Current usage : 1619562496bytes. Limit : 1610612736bytes. Killing task. Dump of the process-tree for attempt_201305041854_0350_m_000000_0 : |- PID PPID PGRPID SESSID CMD_NAME USER_MODE_TIME(MILLIS) SYSTEM_TIME(MILLIS) VMEM_USAGE(BYTES) RSSMEM_USAGE(PAGES) FULL_CMD_LINE |- 30275 20786 30275 30275 (java) 2179 476 1619562496 190241 /usr/jdk64/jdk1.6.0_31/jre/bin/java ...
> Workaround: The workaround is disable all the memory settings (...)

What DISTRIBUTE BY does?

http://stackoverflow.com/questions/18951827/distributed-clause-in-hive

    [1]: http://www.slideshare.net/ye.mikez/hive-tuning
    [2]: https://issues.apache.org/jira/browse/HIVE-6455
