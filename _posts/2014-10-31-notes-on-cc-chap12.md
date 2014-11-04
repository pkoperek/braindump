---
layout: post
title: Notes on "Clean Code - Project Development" chapter
comments: true
---

  * Four rules of _simple_ project:
    1. It passes **all** tests.  
      * A system which is tested throughly and passes the tests each time is a **testable** system.
      * Having a **testable** systems enables its evolution and possibility to refactor/extend it.
    2. Doesn't contain **duplications**.  
      * Improving system structure can involve any kind of good practises
        * Modularization
        * Concern separation
        * Introducing loose coupling, making classes highly cohesive.
    3. Expresses the **intent** of programmer.
      * **Most time of working on a software project is spent on long term maintenance.**
      * The easier to understand the code is, the less time other people need to spend on analyzing it.
      * **Use standard nomenclature**.
    4. **Minimizes** number of methods and classes.
      * In the process of creating small methods and small classes one may create a big number of small classes. This is why that rule also applies to the _number_ of methods and classes (it should be small!).
      * **Big number of classes and methods is a result of pointless dogmatism**. 
      * That rule is the _least_ important out of all presented rules.
  * Rules 2-4 can be applied only when tests are in place.

