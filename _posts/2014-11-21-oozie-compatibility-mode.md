---
layout: post
title: Oozie MR1 compatibility mode
comments: true
---

It took me some time to figure out what does that mean:

    > java.io.IOException: mapreduce.job.outputformat.class is incompatible with reduce compatability mode.
    >   at org.apache.hadoop.mapreduce.Job.ensureNotSet(Job.java:1190)
    >   at org.apache.hadoop.mapreduce.Job.setUseNewAPI(Job.java:1242)
    >   at org.apache.hadoop.mapreduce.Job.submit(Job.java:1279)
    >   at org.apache.hadoop.mapred.JobClient$1.run(JobClient.java:606)
    >   at org.apache.hadoop.mapred.JobClient$1.run(JobClient.java:601)
    >   at java.security.AccessController.doPrivileged(Native Method)
    >   at javax.security.auth.Subject.doAs(Subject.java:396)
    >   at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
    >   at org.apache.hadoop.mapred.JobClient.submitJobInternal(JobClient.java:601)
    >   at org.apache.hadoop.mapred.JobClient.submitJob(JobClient.java:586)
    >   at org.apache.oozie.action.hadoop.MapReduceMain.submitJob(MapReduceMain.java:97)
    >   at org.apache.oozie.action.hadoop.MapReduceMain.run(MapReduceMain.java:57)
    >   at org.apache.oozie.action.hadoop.LauncherMain.run(LauncherMain.java:37)
    >   at org.apache.oozie.action.hadoop.MapReduceMain.main(MapReduceMain.java:40)
    >   at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    >   at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
    >   at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
    >   at java.lang.reflect.Method.invoke(Method.java:597)
    >   at org.apache.oozie.action.hadoop.LauncherMapper.map(LauncherMapper.java:495)
    >   at org.apache.hadoop.mapred.MapRunner.run(MapRunner.java:54)
    >   at org.apache.hadoop.mapred.MapTask.runOldMapper(MapTask.java:428)
    >   at org.apache.hadoop.mapred.MapTask.run(MapTask.java:340)
    >   at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:160)
    >   at java.security.AccessController.doPrivileged(Native Method)
    >   at javax.security.auth.Subject.doAs(Subject.java:396)
    >   at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
    >   at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:155)

Turns out that Oozie (at least in version 3.3.2) runs the map-reduce jobs as if they were designed for MR1. To make it treat your nice-and-shiny MR2 code properly, use those options in configuration for `<map-reduce>` action:

```xml
<!-- New API for map -->
<property>
    <name>mapred.mapper.new-api</name>
    <value>true</value>
</property>

<!-- New API for reducer -->
<property>
    <name>mapred.reducer.new-api</name>
    <value>true</value>
</property>
```
