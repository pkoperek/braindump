---
layout: post
title: Notes on "Clean Code - Classes" chapter
comments: true
---

  * Order of class elements (note no `public` variables on the list :)):
    1. `public static final` constants
    2. `private static` variables
    3. instance variables
    4. `public` methods
    5. `private` methods
  * Uncle Bob allows methods and variables to have default or protected access modifiers for testing purposes. (**Personally - I don't like this idea - it is still breaking the encapsulation**)
  * First rule of classes: **they should be small**.
  * Second rule of classes: **they should be smaller than they are**.
  * **Class should have a single responsibility**
    * **SRP principle: a class should have a single _reason to change_.**
  * Rule of thumb: **if you can't create a cohesive class number - this means the class is too big**.
  * `Processor`, `Manager`, `Super` in name of the class suggests that it has too broad range of responsibility.
  * Rule of thumb: **you should be able to write a short class description - around 25 words, without using words "if", "and", "or", "but"**

    > Each bigger system will contain a lot of business logic code and will be complicated. The basic way to manage this complexity is to organize it in such a way, that the programmer will know where to find specific elements, and at given time he will need to analyze only parts of code directly connected to the problem.

  * Each class should have a small number of instance variables. Each method should use one or more of these variables. The more instance variables are used in the methods the more **cohesive** a class is. Perfectly cohesive class uses all instance variables in all methods.
  * When class looses cohesion - this means that you probably need to split it.
  * Refactoring of legacy code:
    1. Cover the code with unit tests.
    2. Make a small change to the code to improve it.
    3. Run the tests to prove no bugs were introduced.
    4. Go to 2. :)
  * **OCP - Open/Closed principle: classes should be open to extensions and closed to modification**.
  * **DIP - Dependency Injection principle - classes should rely on abstractions - not implementations.**
