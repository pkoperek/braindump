---
layout: post
title: Data Warehouse Toolkit Chapter V - notes
comments: true
---

* Sometimes fact table which seems a monolith at the beginning turns out to contain facts which are not compatible with each other - this requires creating a separate fact table for each identified type. Indicators that this might be the case:
  * Granularity levels.
  * Facts are completely different from each other - they describe different business processes
  * Users want to analyze those groups of facts separately
* **Accumulating snapshot** models something that has **well-defined milestones**.
* Types of dimension changes (_... i have a weird feeling that i already seen that somewhere before ..._)
  * Type 0: retain original - nothing ever changes
  * Type 1: overwrite value in row
  * Type 2: add a new row and reference that new row only in new facts
    * You may want to add additional columns which denote the effective and expiration dates (dates when the value started being used and when it stopped). 
  * Mix of Type 1 and Type 2
  * Type 3: add a new attribute (column) (_waaaat?_) - do this only when the change impacts a lot of rows in dimensions
  * Type 4: Mini-dimension - create a new dimension (with small number of rows) which contains the attributes which frequently change. Unfortunately you need to add a new key to fact table which will refer to the mini-dimension.
  * Type 5: add mini-dimension key to the primary dimension. Overwrite the key reference with each profile change.
  * Type 6: Type 1 (overwrite) attributes in Type 2 (adding new row) dimension (eg. a column denoting the _current_ value of something - there might be a second column which will denote the historic value)
  * Type 7: Dual Type 1 and Type 2 dimensions

It seems that those types are mostly about introducing a combination of volatile (get updated over time) and non-volatile columns which present historical values. If you look at it from hadoop perspective (we can pretty much only add - unless you use hbase) - all types including updates (1,4,5,6,7) don't have much use. 
