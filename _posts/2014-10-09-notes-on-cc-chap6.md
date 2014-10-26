---
layout: post
title: notes on "Clean Code - Objects and Data structures" chapter
comments: true
---

  * Hiding implementation is about **making abstractions** and not putting layers of functions (like numerous getters and setters).
  * "Objects hide the data behind abstractions and expose functions that operate on that data."
  * "Data structures just expose their data and have no meaningful functions."
  * Procedural code makes it easy to **add new functions** without changing the existing data structures. OO code, on the other hand, makes it easy to **add new classes** without changing existing functions.
  * "Mature programmers know that the idea that everything is an object **is a myth**."
  * **The Law of Demeter**  
    Method `f` of class `C` should only call the methods of these:
    * `C` (itself)
    * An object created by `f`
    * An object passed as an argument to `f`
    * An object held in an instance variable of `C`

  * The method should **not** invoke methods on objects that are returned by the allowed functions!
  * If the things which are accessed within a method are data structures with no behavior **the Law of Demeter do not apply**.
  * Avoid creating hybrids (half object - half data structures) - classes which have meaningful methods and expose internal structure via getters/setters.
  * **Refactoring note:** Try to identify what exposed data is used for -> maybe it is possible to extract a method and move it to the exposing class?
    

