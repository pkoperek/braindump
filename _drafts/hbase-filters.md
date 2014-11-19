* What is faster? Scan with a `setStopRow` or Scan with a filter over keys?
* Filters are _pushed down_ to region servers - so computations happen on the cluster

* Suggestion from `RowFilter.java`:

>  * If an already known row range needs to be scanned, use {@link Scan} start
>  * and stop rows directly rather than a filter.

* Scanning with use of filters is slow: [grokbase](http://grokbase.com/p/hbase/user/115cg0d7jh/very-slow-scan-performance-using-filters)
* Answer on Quora: [link](http://www.quora.com/How-feasible-is-real-time-querying-on-HBase-Can-it-be-achieved-through-a-programming-language-such-as-Python-PHP-or-JSP)
* Stack overflow: [link](http://stackoverflow.com/questions/10942638/should-i-user-prefixfilter-or-rowkey-range-scan-in-hbase)
