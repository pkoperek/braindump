---
layout: post
title: Tuning SQL queries a primer
comments: true
---

Goal of tuning SQL queries can be twofold:

* Reducing the turnaround time of query (faster response)
* Reducing the amount of resources included in processing

Classic data tuning optimizes usually for search time of specific rows in a
table (e.g. by creating indexes, limiting the amount of data to search through
by introducing partitions).

Links:

* http://chrismiles.info/systemsadmin/databases/articles/viewing-current-postgresql-queries/
