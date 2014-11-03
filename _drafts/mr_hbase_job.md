1. First problem: external dependencies (jars) for the job. In my case this is: hbase.jar.
   * additional files should get added to distributed cache in job configuration: http://hadoop.apache.org/docs/r2.3.0/api/org/apache/hadoop/filecache/DistributedCache.html

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

   * the output format was missing: job.setOutputFormatClass(NullOutputFormat.class)
   * NullOutputFormat means the job is not returning anything (good for now)

2. Ok, submitting jobs ... (i think): actually starting the job happens inside `job.submit()`.

but throws: InvalidJobConfException: Output directory not set.

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

   * the problem is probably tied to the fact that mentioned hbase table doesn't exist :)

3. Job started working: 

Runned with 
```
$ java -cp "`hbase classpath`test.jar" Prototype
```

```java
public class Prototype {

    private static class MyReadingMapper extends TableMapper<Text, Text> {
        @Override
            protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException {
                // just print out the row key
                System.out.println("Got a row with key: " + Arrays.toString(key.get()));
            }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration configuration = HBaseConfiguration.create();

        Job job = Job.getInstance(configuration);
        job.setJarByClass(MyReadingMapper.class);

        Scan scan = new Scan();
        scan.setCaching(500);
        scan.setCacheBlocks(false);

        // table name should be parametrized
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
  * spaws 1 mapper per regionserver (that can be a lot)
  * although i didn't create a reducer - something is going on that side

4. I wanted to use MultipleOutputFormat (to output multiple files from the job). Turned out that this API is not supported anymore in Hadoop2. You need to use MultipleOutputs. But this solution has an advantage: I might not need reducers at all!
  * http://stackoverflow.com/questions/15100621/multipletextoutputformat-alternative-in-new-api
  * http://stackoverflow.com/questions/20209060/hadoop-multipleoutputformat-support-for-with-org-apache-hadoop-mapreduce-job
  * http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapreduce/lib/output/MultipleOutputs.html

5. Output directory exists:

Exception in thread "main" org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory hdfs://athena/user/test/prototype already exists
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

  * The solution is to extend the TextOutputFormat class as instructed on http://mail-archives.apache.org/mod_mbox/hadoop-mapreduce-user/201204.mbox/%3CCA0E4C80784FAE4398333564344FDB8F997E3BFBF6@EXCHANGE08.webde.local%3E

