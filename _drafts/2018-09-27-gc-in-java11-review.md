---
layout: post
title: Modern Java Review - Garbage Collection techniques
comments: true
---

This is this time of the year: I want to brush up my Java knowledge! Since it
has been a while I was using it every day I thought it would be good that this
time I go deep on the new features: 

* new garbage collection algorithms
* lambda & functional programming in JVM
* Stream API
* syntactic sugar which might have already been there for a while since Java 7

This is the first post in a series of I hope at least four. Without further
ado, lets dig in into the first topic: **Garbage Collection**.

## Garbage Collection

### Z Garbae Collector (ZGC) - new in Java 11

* scalable low latency garbage collector
* very low pause times (<10 ms) on multi-terabyte heaps
* limited to 4TB of data at heap
* uses new techniques
  * coloured pointers
  * load barriers

#### Coloured pointers

* on 64 bit systems a pointer can address vastly more RAM than the system can
  realistically provide - this means some of the bits can be used to store some
  additional information
* ZGC is limited to 4Tb heaps - it needs only 42 bits for addressing
* the rest - 22 bits are used to store some state (currently 4 bits occupied)
  * finalize 
  * remap
  * mark0
  * mark1

Links:

* https://www.opsian.com/blog/javas-new-zgc-is-very-exciting/

### G1

* https://wiki.openjdk.java.net/display/zgc/Main
