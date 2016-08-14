---
layout: post
title: Data Warehouse Toolkit Chapter XXI notes
comments: true
---

* First build stuff in a sandbox environment - productionize when they are successful 
* Try with simple applications first
* In general do the same as with RDBMS:
  * Drive the data source with business needs
  * Focus on user usability and good-enough performance
  * Think dimensionally
  * Use conformed dimensions
  * Use _Slowly Changing Dimensions_ techniques
  * Use surrogate keys

* Don't do ad hoc querying - rather focus on analytics
* Use clouds for prototyping
* Search for performant, highly tuned dedicated solutions for your specific use-case
* Use hadoop for integration of many sources of data (possibly some of them are unstructured)
