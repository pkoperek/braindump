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


    [1]: http://www.slideshare.net/ye.mikez/hive-tuning
    [2]: https://issues.apache.org/jira/browse/HIVE-6455
