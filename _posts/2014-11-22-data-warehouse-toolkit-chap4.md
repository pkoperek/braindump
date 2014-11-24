---
layout: post
title: Data Warehouse Toolkit Chapter IV notes
comments: true
---

* Three ways of modelling inventory:
  * Daily snapshots with amounts _on hand_ and _sold_.
  * Record all transactions which affect inventory (but it is hard to use for analytics)
  * Accumulating snapshot: 1 row denotes the whole lifecycle of an item (eg. there is a column with a date when arrived in warehouse, date when was inspected and row when leaves the warehouse - this row is updated over a longer timespan).

* **Enterprise Data Warehouse Bus Matrix** - the whole thing just means that different business processes contribute to different dimensions. Some of the dimensions are common across the company. If you're implementing a warehouse dear reader - just do it one process at a time.
* Author emphasizes that the "bus matrix" is a tool for communication and work organization (you can use it to tell someone what is being implemented and you can plan that eg. you implement now that particular row).
* Don't overgeneralize - employees != customers (although in physical world both are people... well at least usually ;))
* Ideally - use the same dimension tables across all fact tables (from different business processes)* Unfortunately - world isn't ideal (mostly...). It is also acceptable to have the dimensions conform maybe not fully - but in a subset of features:
  * Dimension for one of the fact tables contains a subset of other dimension's columns (eg. a **Product Dimension** may contain _Brand Description_, _Category Description_, _Subcategory Description_ and a **Brand Dimension** may contain only _Brand Description_ and _Category Description_. Those tables have the same values in the subset of columns: _Brand Description_ and _Category Description_. This is what makes them conformed.
  * Dimension which contains a subset of rows of another dimension. 
* Sometimes it doesn't make sense to integrate everything (eg. each department might have own customers and there is no will to do a cross-sell deal).
* Fact tables also need to conform. 

> You must be disciplined in your data naming practises. If it is impossible to conform a fact exactly, you should give different names to the different interpretations so that business users do not combine these incompatible facts in calculations.
