---
layout: post
title: faking DUAL table in hive
---

It is sometimes beneficial to have something constant in database. RDBMs engines like Oracle or DB2 have tables like DUAL or SYSIBM.SYSDUMMY1. In hive there is no such thing by default ... But why not create a custom one? The easiest way (which I think works on all (???) Linux boxes) is to create one based on `/etc/hostname`. 

```sql
CREATE OR REPLACE TABLE dual (dummy STRING);
LOAD DATA LOCAL INPATH '/etc/hostname' OVERWRITE INTO TABLE dual;

INSERT OVERWRITE TABLE dual
SELECT
    "X" AS dummy
FROM
    dual
LIMIT 1;
```

Hacky... But on the other hand pretty simple.
The other way to do it is ... custom UDTF. Check how it can be done with this project: https://github.com/pkoperek/hive-single-row-udtf 

Oracle docs on `DUAL`: http://docs.oracle.com/cd/B19306_01/server.102/b14200/queries009.htm
