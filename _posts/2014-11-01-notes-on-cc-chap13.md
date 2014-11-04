---
layout: post
title: Notes on "Clean Code - Concurrency" chapter 
comments: true
---

  * Writing clean parallel programs is **hard**.
  * Concurrency helps to split _what_ is being executed from _when_ it is executed.
  * Concurrency introduces overhead - both on performance and writing code.
  * Concurrent errors are hard to trace and often hard to reproduce.
  * Concurrency often requires changing primary rules of a project.
  * Number of critical sections should be as small as possible.
  * Size of critical sections should be as small as possible.
  * Suggestions:
    * Code related to concurrency should be split from business logic.
    * Access to data, which can be shared, should be limited with hermetization.
    * It is worth trying to split data in independent subsets, which can be separately processed by independent threads (possibly on separate processors).
    * Since 1.5, Java has a nice built-in library for dealing with common parallel processing problems - learn how to use it.
    * Write tests which can expose concurrency problems - run them as often as possible on a broad number of machines, operating systems, configurations etc. 
    * When debugging, ensure that business logic (single-threaded code) works correctly.
    * Write parallel code which can be configured and test how it works in many different configurations.
    * Don't ignore single system failures - they are probably just hard-to-reproduce problems. 
    * Enhance code with elements which can introduce failures. (_what about introducing elements on the infrastructure level which can deliberately cause problems and test whether the whole system is resilient? Simian Army to the rescue!_)
  * Ideal way of eliminating problems with shared data is _eliminating_ data sharing (eg. by copying whole structures and treating them as read-only).
  * Threads should be as **independent** as possible.
  * Dependencies between synchronized methods can introduce hard-to-trace problems - it is suggested to **make a single synchronized method per class**.
  * **Testing doesn't guarantee correctness** - it just minimizes the risk of writing incorrect software.
  * Concurrent code should be **tuneable**.
  * Run more threads than you have processors - it increases the probability of detecting deadlocks.
  * Concurrent code should be executed often on all target platforms.
