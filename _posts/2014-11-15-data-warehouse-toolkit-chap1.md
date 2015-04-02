---
layout: post
title: The Data Warehouse Toolkit Chapter I - notes
comments: true
---

* Data Warehouses are analytical tools - based on them business is going to make decisions.
* It doesn't have any sense to create a warehouse if nobody wants to use it.
* Warehouse needs to be:
  * understandable by business people - using business vocabulary
  * **trustworthy**
  * consistent
  * easily accessible
  * adaptable (needs to change with business needs)
  * secure
  * fast enough 
* **Star schema** - dimensional model implemented in RDBMS.
* **Fact == bussiness measure**
* Fact table - stores the performance measurements result from a business event. All measurement rows must be on the same grain (level of detail). **This is a bedrock principle for dimensional modeling.**
  * **The most useful facts are numeric and additive** - it is possible to carry out aggregations on them.
  * There are semi-additive facts (eg. account balance) - can be summed in general, but eg. not in time dimension.
  * _Facts are often described as continuously valued_ (dimensions can be discrete)
  * Include only true activity (don't fill fact tables with zeros - those zeros will probably overwhelm the whole table). 
  * Transaction grain fact tables are the most common.
  * Fact tables have foreign keys to dimension tables.
  * Fact table has own primary key composed of **a subset of the foreign keys**. This key is called a _composite key_.
  * _Every table having a composite key is a fact table_ - all other are dimension tables.
* Dimension tables contain _context_ associated with the measurement event. 
  * They describe the **"who, what, where, when, how, and why"**.
  * They have many columns/attributes (even up to 100!).
  * Have fewer rows than fact tables - but are wide.
  * Have a single **primary key**.
  * Are a primary source of query constraints, groupings, report labels.
  * Attributes should be described with actual human-understandable names 
  * _In many ways, the data warehouse is only as good as the dimension attributes_.
  * Rule of a thumb: 
    > Continuously valued numeric observations are almost always facts; discrete numeric observations drawn from a small list are almost always dimension attributes.
  * Almost always dimension table space should be traded off for simplicity and accessability.

* Each business model should be modelled with its own star schema model.
* Kimball's DW/BI architecture
  * Comprises of four components:
    * Operational source systems - systems the data is copied from
    * ETL system - everything between presentation area and operational systems; basically tools which gather the data and pre-process it (_extract_), clean, combine multiple sources, de-duplicate, etc (generally speaking _transform_) and transfer data to target physical storage (_load_).
* Using a 3NF structure for ETL processing doesn't have much sense - it would require to do a single ETL process from operational to that format, and then again extract-transform-load into presentation space.
    * Presentation area (supporting business intelligence) - it is all BI sees from warehouse project. The data available to users should be fine-grained. It is necessary to be able to use the warehouse for ad-hoc queries.
      * All dimensional structures need to be built using common, conformed dimensions.
    * BI applications - from query tools to machine learning algorithms.

* Other (than Kimball's) architectures:
  * Data mart per company department (large duplication: of data, processes and funding).
  * Corporate Information Factory - ingest data to Enterprise Data Warehouse (normalized to 3NF) and later pump it to individual departments' data marts (summaries/aggregated data). EDW with fine-grain data is available to users. 
  * Hybrid - pretty much the same as CIF - but this time users can't use EDW - atomic data is pushed to individual marts. 

* "Pearls of wisdom":
  * Data needs to be on the most detailed level - summaries will never be able to answer all questions.
  * Dimensions should be organized around business processes - not around departments.
  * Dimensional models are scalable (? no evidence provided).
  * Dimensional models are for ad-hoc usage (aggregation, summing etc - only for performance reasons and addition to raw data).
  * Need to rely on conformed naming schemes to be able to integrate dimensional models with outside tools.



