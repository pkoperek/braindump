---
layout: post
title: The Data Warehouse Toolkit - Chapter III - notes
comments: true
---

Chapter is a use case of applying dimensional modelling in a typical retail organization scenario.

1. Select business process
   * Sales transactions at POS (_Point Of Sale_). 
2. Declare the grain
   * Grain level == single product on a POS transaction.
3. Identifying dimensions:
   * Date of sale (**when**)
   * **Where** the sale occurred 
   * Promotions (context)
   * Cashiers (context)
   * Method of payment (context)
4. Identify facts (_may require changing grain!_). Note that they are mostly nicely additive
   * Sales quantity
   * Per unit regular price
   * Discount
   * Net paid prices
   * Extended discount
   * Sales dollar amount
   * Extended dollar amount (if provided)
   * Extended cost amount (if provided)
   * Extended ...
   * Quote: _"Percentages and ratios, such as gross margin, are non-additive."_

5. Next estimate the size of data inflow (rows per day/week/month) to fact table.
6. Define dimension table attributes 
   * Columns in date dimension table can include the regular date info, fiscal year/month/date info, date exploded to some particular format, quarter indicator (pretty much anything which enables querying by some specific date conditions).
   * Use textual attributes instead of flags and indicators (_Holiday_, _Non-holiday_ instead of _Y/N_).
   * Products dimension illustrates the "dimension hierarchy within table" technique. Instead of creating a complicated table structure - just duplicate the data in several columns (brand, category, etc)
   * If product key contains embedded information - explode it to separate columns.
   * Rule of a thumb when in doubt what is fact or dimension attribute:

   > Data elements that are used both for fact calculations and dimension constraining, grouping, and labeling should be stored in both locations.

* Kimball suggests that derived data such as gross profit (diff between cost and sales price) should be computed at ETL stage and made available to users. Don't think it is a good idea - I would prefer the solution with view and dynamic computation of this kind of value. On the other hand depending on what is the actual data set size is it might be the only option. 
* **Drill down** - just add a column to report, which slices already presented rows in more detailed categories.
* **Degenerate dimension** - dimension without a table. Exists just as a column in fact table. Can be used for grouping facts or just as a back reference to operational system.
* Adding new dimension: create a table, add a single row "Prior to introducing new dimension", add a new column to fact table, populate it with key to that "gravestone" row.
* Sometimes it is necessary to introduce a _factless_ fact table - eg. to describe what _didn't_ happen. Example with promotions: we want to know whether promoted products were sold during promotion. Sales fact table doesn't contain that - we store only info about what was actually sold. Solution: create a new fact table (artificially) which will contain the number of product units sold per day. A bit aggregatish - but for specified use should be ok.
* Dimensions primary keys should be _surrogate keys_ instead of _natural keys_.
  * They protect the whole warehouse from operational changes (if natural keys changes - you still have your own).
  * Enable integration of different source systems which might potentially clash keys. The other observation: the same fact in different sources can have a different natural key!
  * Performance (_nobody should need more than an int as a dimension table key_ - LOL :))
  * It is easier to handle natural key null values.
  * Can handle changes of dimension tables gracefully.
* Dimension natural keys should be stored as a separate attribute in dimension table
* **Fact table key** is often a subset if table's foreign keys (to dimension tables) and a degenerate dimension. 
  * Having the surrogate key as primary key in fact table has many benefits (technical - not analytical)
    * Checkpoint in bulk load
    * Immediate unique row identification
    * **Replace updates with row insert/delete** - _now this is something useful for hive @ HDFS_

Final thoughts:

  * don't do "snowflakes" in dimensions (don't normalize dimensions) - doesn't make much sense and can actually harm performance (joins!)
  * outriggers (referencing dimensions in dimensions) are permissible - but not advised
  * avoid using too many dimensions in a single fact table - 20/25 tops. If you have more - try to combine correlated dimensions into one.
