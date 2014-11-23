---
layout: post
title: Data Warehouse Toolkit Chapter VIII notes
comments: true
---

* Customer contact table:
  * Split it down to as many elemental parts as possible.
* Use unicode as encoding - otherwise handling multiple encodings will be a nightmare
* If warehouse is to be used internationally - translate the reports, and not all content of warehouse
* According to authors, data mining teams should get exports from warehouse, so they can do computations on their own servers. (_imagine exporting couple hundred TB :)_)
* Solution for tracking user's activity - create a "steps" dimension table which will refer to what user can do. Then link each event to that dimension.
