---
layout: post
title: hive <=> operator 
comments: true
---

Human being learns all the time ... I've just found out that there is a wonderfully simple way of writing equal conditions including NULL values in Hive: [the <=> operator](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF). 
I used to write something like this:

```sql
...
WHERE 
  column1 = column2 OR (column1 IN NULL AND column2 IS NULL) 
```

Now it boils down to:

```sql
WHERE
  column1 <=> column2
```

For reference: [NULL = NULL is false by definition in SQL](http://en.wikipedia.org/wiki/Null_%28SQL%29)
