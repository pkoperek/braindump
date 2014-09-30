---
layout: post
title: writing udtfs in hive
---

UDTF = User Defined T Function

Need to implement GenericUDTF interface
Can output more than 1 row for each input row.


Four crucial methods:

* `initialize` - informs about the format of input data (type validation should be carried out here). The returned value is a `ObjectInspector` which describes the output row format (basically the object which is passed to `forward` method).
* `process` - process actual input rows (may not be invoked if no data passed to function)
* `close` - notification from hive that there is no more data to process - at this point final results should be returned. *Note: this method is called even if there were no input rows - by returning results only here you can create a function which always generates the same number of rows*
* `forward` - call this method to return a row of results (can be called from `process` and `close`)
