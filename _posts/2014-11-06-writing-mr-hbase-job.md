---
layout: post
title: Writing MR job which reads (scans a table) from HBase
comments: true
---

HBase allows efficient reads and writes of single rows (as opposed to hive which is better at processing them in bulk). Unfortunately - sometimes it is actually required to read and process a bunch of rows together (eg. to compute some aggregates). There are couple things which could be done:

  * create an external table in hive which would be backed up by HBase (queries will run - but it will be _very_ slow)
  * use tools like [Phoenix][1] (no idea how it works in terms of performance)
  * write your custom MR job which does the processing - this post is exactly about it :)

I was writing my code from scratch: below a list of problems which I encountered and managed to workaround or find a decent fix.

  1. Adding required external dependencies (jars) for the job. 

    In my case this was the `hbase.jar`. According to official [documentation][2] additional files should get added to distributed cache in job configuration. The actual solution is quite nice: `TableMapReduceUtil.initTableMapperJob` and `TableMapReduceUtil.initTableReducerJob` add this file automatically. 

  2. Exception:

    ```
Exception in thread "main" org.apache.hadoop.mapred.InvalidJobConfException: Output directory not set.
    at org.apache.hadoop.mapreduce.lib.output.FileOutputFormat.checkOutputSpecs(FileOutputFormat.java:133)
    at org.apache.hadoop.mapreduce.JobSubmitter.checkSpecs(JobSubmitter.java:433)
    at org.apache.hadoop.mapreduce.JobSubmitter.submitJobInternal(JobSubmitter.java:335)
    at org.apache.hadoop.mapreduce.Job$11.run(Job.java:1286)
    at org.apache.hadoop.mapreduce.Job$11.run(Job.java:1283)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:396)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
    at org.apache.hadoop.mapreduce.Job.submit(Job.java:1283)
```

    The problem was that the output format was missing. Adding `job.setOutputFormatClass(NullOutputFormat.class)` fixed the issue. Using `NullOutputFormat` means that the job will not return anything _(at that point I didn't want to emit anything. Later on I actually specified a different output format - which stored the data properly in HDFS.)_

  3. `job.submit()` throws again:

    ```
Exception in thread "main" java.io.IOException: No table was provided.
    at org.apache.hadoop.hbase.mapreduce.TableInputFormatBase.getSplits(TableInputFormatBase.java:152)
    at org.apache.hadoop.mapreduce.JobSubmitter.writeNewSplits(JobSubmitter.java:468)
    at org.apache.hadoop.mapreduce.JobSubmitter.writeSplits(JobSubmitter.java:485)
    at org.apache.hadoop.mapreduce.JobSubmitter.submitJobInternal(JobSubmitter.java:369)
    at org.apache.hadoop.mapreduce.Job$11.run(Job.java:1286)
    at org.apache.hadoop.mapreduce.Job$11.run(Job.java:1283)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:396)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
    at org.apache.hadoop.mapreduce.Job.submit(Job.java:1283)
```

    The problem is probably connected with the fact that mentioned test table doesn't exist :) After creating it in hbase the problem vanished.
    At this point I got some sample code which started working: 

    ```java
    public class ReadOnly {
      private static class MyReadingMapper 
                           extends TableMapper<Text, Text> {
      
        @Override
        protected void map(ImmutableBytesWritable key, Result value, 
                           Context context) 
                           throws IOException, InterruptedException {
        // just print out the row key
        System.out.println("Got a row with key: " + 
                           Arrays.toString(key.get()));
        }
      }
  
      public static void main(String[] args) 
                    throws IOException, 
                           ClassNotFoundException, 
                           InterruptedException {

        Configuration configuration = HBaseConfiguration.create();

        Job job = Job.getInstance(configuration);
        job.setJarByClass(MyReadingMapper.class);

        Scan scan = new Scan();
        scan.setCaching(500);
        scan.setCacheBlocks(false);
        
        TableMapReduceUtil.initTableMapperJob(
                "test_table",
                scan,
                MyReadingMapper.class,
                Text.class,
                Text.class,
                job
                );
        job.setOutputFormatClass(NullOutputFormat.class);
        job.submit();
        job.waitForCompletion(true);
      }
    }
```

    Observations: 
    * Job spaws 1 mapper per Region Server
    * Although I didn't configure reduction step at all, some reducers were kicked off.

  4. I wanted to use `MultipleOutputFormat` (to output multiple files from the job). Turned out that this API is not supported anymore in Hadoop2. You need to use `MultipleOutputs`. But this solution has an advantage: I might not need to confiigure reducers at all! Some useful docs: [here][3], [here][4] and [here][5].

  5. This time hadoop attacked with: `Output directory exists` exception:

    ```
Exception in thread "main" org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://myhadoop/user/test/prototype already exists
    at org.apache.hadoop.mapreduce.lib.output.FileOutputFormat.checkOutputSpecs(FileOutputFormat.java:141)
    at org.apache.hadoop.mapreduce.lib.output.FilterOutputFormat.checkOutputSpecs(FilterOutputFormat.java:61)
    at org.apache.hadoop.mapreduce.lib.output.LazyOutputFormat.checkOutputSpecs(LazyOutputFormat.java:83)
    at org.apache.hadoop.mapreduce.JobSubmitter.checkSpecs(JobSubmitter.java:433)
    at org.apache.hadoop.mapreduce.JobSubmitter.submitJobInternal(JobSubmitter.java:335)
    at org.apache.hadoop.mapreduce.Job$11.run(Job.java:1286)
    at org.apache.hadoop.mapreduce.Job$11.run(Job.java:1283)
    at java.security.AccessController.doPrivileged(Native Method)
    at javax.security.auth.Subject.doAs(Subject.java:396)
    at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
    at org.apache.hadoop.mapreduce.Job.submit(Job.java:1283)
```

    The solution was to extend the `TextOutputFormat` class as instructed [here][7] and use the new version as mapper's output format. Afterwards the job was properly spitting out files to multiple directories according to hbase row content. Phew...

In the whole process of writing the job the [HBase manual][6] turned out to be very helpful.

[1]: http://phoenix.apache.org/
[2]: http://hadoop.apache.org/docs/r2.3.0/api/org/apache/hadoop/filecache/DistributedCache.html
[3]: http://stackoverflow.com/questions/15100621/multipletextoutputformat-alternative-in-new-api
[4]: http://stackoverflow.com/questions/20209060/hadoop-multipleoutputformat-support-for-with-org-apache-hadoop-mapreduce-job
[5]: http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapreduce/lib/output/MultipleOutputs.html
[6]: http://hbase.apache.org/book/mapreduce.example.html
[7]: http://mail-archives.apache.org/mod_mbox/hadoop-mapreduce-user/201204.mbox/%3CCA0E4C80784FAE4398333564344FDB8F997E3BFBF6@EXCHANGE08.webde.local%3E

