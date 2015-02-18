---
layout: post
title: Spark fun - Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient memory 
comments: true
---

I've hit that problem after trying to update from Spark 1.1.0 to 1.1.1 (and 1.2.0). It was really confusing - at least 1.1.1 was supposed to be backwards compatible. I didn't change the configuration and the only other part of the system which was different than previously was mesos 0.20 -> 0.21. In fact mesos triggered the change since the offer format changed. Checking the mesos/spark UI didn't reveal anything wrong - in fact the workers had more memory than I requested (2,7GB and 2,5GB respectively). And so the digging begun...

Google didn't show anything interesting in particular:

  * [Spark @ Hortonworks YARN](http://hortonworks.com/hadoop-tutorial/using-apache-spark-hdp/)
  * [Unanswered spark-user post](http://mail-archives.apache.org/mod_mbox/spark-user/201501.mbox/%3CCAJOeOZ6Uzq2wQr_UwYmQLqiUnup7+5ugWQwL_0Q_euQU=zMBmg@mail.gmail.com%3E)
  * [Blog post indicating actual memory configuration problem](http://dandydev.net/blog/spark-initial-job-resources)
  * [Not started Spark nodes](https://groups.google.com/forum/#!topic/predictionio-user/Bq0HBCM1ytI)
  * [Bad worker memory configuration](http://community.cloudera.com/t5/Advanced-Analytics-Apache-Spark/TaskSchedulerImpl-Initial-job-has-not-accepted-any-resources/td-p/8732)
  * [... even a stackoverflow thread](http://stackoverflow.com/questions/21677142/running-a-job-on-spark-0-9-0-throws-error)

Network connectivity was fine (checked if ports open, machines pinging, dns resolution etc). Then I started observing worker nodes (to check who has the problem - Spark Master or workers) and it turned out ... executors did not start at all. Spark binary wasn't even downloaded. Maybe `mesos-slave` configuration did in fact change? I started reading again the [Mesos installation guide](http://mesosphere.com/docs/getting-started/datacenter/install/#verifying-installation). Maybe slaves weren't able to contact master or receive executor configurations? 

It is actually very simple to test (this is installation verification step from the guide):
```
MASTER=$(mesos-resolve `cat /etc/mesos/zk`)
mesos-execute --master=$MASTER --name="cluster-test" --command="sleep 5"
```

Quick look at mesos UI - it's there! Ok so this begins to be a purely problem of Spark scheduling tasks on mesos. Bring on the source code!

After some digging I found the `MesosSchedulerBackend` class which contains a very interesting method: `resourceOffers`. Basically this is the place where resource offers from mesos are being evaluated/accepted/rejected by Spark. Here it gets interesting... In version 1.1.0 there is such a condition:

```scala
def enoughMemory(o: Offer) = {
  val mem = getResource(o.getResourcesList, "mem")
  val slaveId = o.getSlaveId.getValue
  mem >= sc.executorMemory || slaveIdsWithExecutors.contains(slaveId)
}
```

which changes in 1.1.1 [SPARK-3535](https://issues.apache.org/jira/browse/SPARK-3535) to:

```scala
def sufficientOffer(o: Offer) = {
  val mem = getResource(o.getResourcesList, "mem")
  val cpus = getResource(o.getResourcesList, "cpus")
  val slaveId = o.getSlaveId.getValue
  (mem >= MemoryUtils.calculateTotalMemory(sc) &&
    // need at least 1 for executor, 1 for task
    cpus >= 2 * scheduler.CPUS_PER_TASK) ||
    (slaveIdsWithExecutors.contains(slaveId) &&
    cpus >= scheduler.CPUS_PER_TASK)
}
```

Now there are significant differences! First of all 1.1.1 takes number of cpus into account. If You don't have a spare core for mesos-executor process the offer is not accepted. This means that machines with a single core have no chances of being used. I was using `m1.medium` instances - **bang that is the issue!** The second important thing: the way of computing the amount of required memory is different. In 1.1.0 anything what had more than `spark.executor.memory` was accepted. In later versions it needs to satisfy this condition:

```scala
private[spark] object MemoryUtils {
  // These defaults copied from YARN
  val OVERHEAD_FRACTION = 1.07
  val OVERHEAD_MINIMUM = 384

  def calculateTotalMemory(sc: SparkContext) = {
    math.max(
      sc.conf.getOption("spark.mesos.executor.memoryOverhead")
        .getOrElse(OVERHEAD_MINIMUM.toString)
        .toInt + sc.executorMemory,
        OVERHEAD_FRACTION * sc.executorMemory
      )
  }
}
```

This means You need to have more memory than `spark.executor.memory * 1.07` and `spark.executor.memory + spark.mesos.executor.memoryOverhead (default: 384)`. 

Fortunately for me: all this can be tweaked with configuration. I tuned the memory configuration with `spark.mesos.executor.memoryOverhead` and it works in the same way as previously. Regarding cpu cores:
since [nibbler](https://github.com/pkoperek/nibbler) is creating long-running mesos executors I just switched to [coarse resource mode](http://spark.apache.org/docs/1.2.0/running-on-mesos.html#mesos-run-modes) - `spark.mesos.coarse=true`. In this mode spark accepts all resources which have at least 1 core (;)) what is all I want! Yay Spark! :)


