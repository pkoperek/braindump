---
layout: post
title: draft
---

Spell checking in Vim:

```
:set spell spelllang=en_us
```

DROP TABLE IF EXISTS test_avro;
CREATE TABLE test_avro
ROW FORMAT SERDE
'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT
'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT
'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
TBLPROPERTIES (
'avro.schema.url'='hdfs:///user/koperek/schema.json');
LOAD DATA LOCAL INPATH "avro_data_file.avro" OVERWRITE INTO TABLE test_avro;
select * from test_avro;

