
http://grokbase.com/t/cloudera/impala-user/1455kjz4d7/impala-udf-functions-runs-even-after-jar-is-removed

BUSTED

Tried with a Java UDF: same jar name, same class names, same UDF names and signatures. Version of Imapala used: 1.4.1.

Tried also a different scenario: two jars with the same name (but different paths in HDFS), same class names, two different implementattions, each implementation registered in different schema. Both implementations are executed correctly.
