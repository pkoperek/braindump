---
layout: post
title: draft
---

Hive 0.10:
Shitty exception not saying what is going on:

```
FAILED: SemanticException [Error 10025]: Line 3:0 Expression not in GROUP BY key 'XXXXXX'
```

The problem is ... the custom UDAF was not created (with CREATE TEMPORARY FUNCTION). 
Turns out hive doesn't realize that the function is missing and assumes it isn't a UDAF - so instead to just say that the function is missing it craps out with random problem.


-----

In init() method always add a super.init(...) cause without that aggregate() method doesn't work
