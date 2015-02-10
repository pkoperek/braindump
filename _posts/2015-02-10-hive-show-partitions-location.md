---
layout: post
title: Showing partition location in hive
comments: true
---

Easy to do but still I spent some time on figuring it out... 

```sql
  SHOW TABLE EXTENDED LIKE <table_name> PARTITION ( <partition_spec> ); 
```

`<table_name>` is actually a pattern - you can use wildcards!

Links:
  * [Hive manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ShowTable/PartitionExtended)
  * [Stack Overflow thread](http://stackoverflow.com/questions/19556166/how-to-know-location-about-partition-in-hive)
