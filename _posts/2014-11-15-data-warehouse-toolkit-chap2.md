---
layout: post
title: The Data Warehouse Toolkit Chapter II - notes
comments: true
---

* Fundamental concepts of dimensional modelling:
   1. Gather requirements and needs of the business.
   2. Before designing anything - talk with experts (_workshops_ - WAAT?).
   3. Design the dimensional model:
      1. Select process.
      2. Define the granularity level.
      3. Identify dimensions.
      4. Identify facts.
   4. Business process is _something_ done by the organization - one execution of a process is usually a single row in fact table.
   5. Grain (granularity) - define *what* is a single row in a table (_atomic grain_ - lowest level at which data is captured within the business process - can't be split up; _atomic grain - food for atomic zombies_).
   6. Identify the context - which is enclosed in dimensions.
   7. Identify facts - which are almost always numeric and usually correspond some physical action.

* **Star schema = dimensional structure deployed in a RDBMS.**
* **OLAP cube - dimensional structure implemented in a multidimensional database.**
* Facts can be additive (can be summed across _all_ dimensions related to the table), semi-additive (can be summed across _some_ dimensions), non-additive.
* Instead of putting `NULL` values into FKs in fact table - create **guard** rows in dimension tables.
* Dimension table primary key can't be the operations system's natural key - it needs to be artificial, because there will be **multiple dimension rows** for that natural key (**dimension rows changes are tracked over time**).
* Dimension tables can contain multiple hierarchies (eg. for products: categories, brands etc)
* Dimensions can be used multiple times in the same fact table row in different contexts. It is recommended to create views for dimension table to make the context explicit.
* Merge low-cardinality flags/indicator dimensions into a single dimension table (less joins!). Don't create cross product of all combinations - create rows only for combinations which are actually used.
* Avoid normalizing dimension tables (multilevel structure of _snowflake_) - it is difficult for business users to understand it.
* _Conformed Dimensions_ - dimensions which are used across different fact tables - can be used to join data from different business processes.
* Changing dimension attributes:
  1. No need to change - attributes never change (!)
  2. Overwrite - old row gets overwritten with new value
  3. Add new row - old fact rows still refer to old rows - thus history is preserved
  4. Add new attribute/column to dimension table (_alternate reality_) - the old value is still visible and can be easily used in the same time with the new one
  5. Create a mini-dimension - containing the frequently changing _attributes_ within a dimension table. Mini-dimension has own primary key - this key is stored in fact table. (_table with 2 primary keys?_)
  6. Combination of overwriting, adding new rows etc - based upon a specific need. 

* Dimension hierarchies:
  * Create column per hierarchy level.
  * Create bridge table (which can represent the hierarchy).
  * Create a string attribute which contains the path in hierarchy
  * Don't create separate dimensions for each level of hierarchy! 

* Creating `audit` dimensions is very useful and makes debugging easier.
* _Abstract Generic Dimensions_ - avoid creating generic dimensions which eg. contain all information for location for location in warehouse, location in country etc. It most probably will have negative impact on performance.
* In case fact create hierarchies: create one _core_ table which can be used to store common information, and separate fact tables for each subtype.
* Create a fact table for error situations: eg. if there is an error during processing, a row describing what failed should be added.
