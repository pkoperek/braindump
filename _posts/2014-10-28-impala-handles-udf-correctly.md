---
layout: post
title: MythBusters Impala edition "Impala udf functions runs even after jar is removed" BUSTED
comments: true
---

While I was working on a Java Impala UDF, a colleague pointed me to: [Impala udf functions runs even after jar is removed](http://grokbase.com/t/cloudera/impala-user/1455kjz4d7/impala-udf-functions-runs-even-after-jar-is-removed)

Thought that should give it a try - it might happen that all my work will be useless because of that. I prepared two jar files with the same class names (but different implementations!). UDF signatures were the same. The scenario was as follows:

  1. Create a function from first jar in schema A.
  2. Execute query with that function.
  3. Drop function.
  4. Try to execute query with function (failed - as expected).
  5. Replace the first UDF jar file in HDFS with second implementation.
  5. Create a function from the same path in HDFS in schema A (with the same UDF signature!).
  6. Execute query with that function - query worked and second version of code was invoked.

No signs of problems described in the original post. Version of Imapala used: **1.4.1.** 
I've also tried a different scenario: 
  
  * two jars with the same name (but different paths in HDFS)
  * same class names
  * two different implementattions
  * each implementation registered in different schema

Unfortunately - in this case it also worked fine ;)
