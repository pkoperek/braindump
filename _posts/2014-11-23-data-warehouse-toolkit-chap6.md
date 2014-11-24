---
layout: post
title: Data Warehouse Toolkit Chapter VI - notes
comments: true
---

* Avoid normalizing fact tables - in most situations it is far better to have some duplication and a simple schema than no redundant data but very complex schema.
* Instead of using a single date dimension table it might be better to create _one physical_ table and _multiple views_ - each for a different context. (Kimball call this _role playing_)
* Product dimension guidelines:
  * Avoid normalizing product dimension table (rather than using "snow flakes" just create one wide table with duplicated data inside)
  * Don't use the operational product key - use a surrogate key
  * Instead of operational codes used in operational database - use descriptive attribute values (eg. instead product type MGHM use "Magical Hat - Color Magenta"
  * Do a quality check which rule out misspellings, impossible values etc.
  * Document all assumptions in the metadata (_table comments?_)
* Customer dimension guideline
  * Avoid creating geographical hierarchies :)
* Junk dimension - sometimes there is a bunch of indicators/flags/enumerations which don't fit anywhere use - you can create one table which will include them all and just reference to a row with combination of those indicators.
* There should be a orders of magnitude difference in size between dimension and fact tables.
* Strategies for storing multiple currencies:
  * Duplicate fact rows in different currencies
  * Hold converted currencies in columns 
  * Create a fact table with daily currency conversions.
* Don't join fact tables! 
* If you need to store the same information in different units:
  * Think about adding already converted values as columns
  * If there are too many columns which would have to be converted - add the conversion factor as column
