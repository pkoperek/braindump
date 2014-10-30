---
layout: post
title: Notes on "Clean Code - Systems" chapter
comments: true
---

  * _Dependency Injection_ (using _Inversion of Control_ to manage dependencies) splits construction of objects from places where they are used **(this helps with preserving Single Responsibility Principle)**.
  * While using _dependency injection_ the user-class (the client) exposes only constructor parameters or setter methods. They are later used to **inject** the dependencies.
  * _It is impossible to build the system properly the first time. Instead of that, it is much safer to implement what is actually required and rebuild and extend the system along the way. Iterative approach is essential._

  > Software systems are different than physical systems. Their architecture can be expanded iteratively, **if** you separate concerns well.

  * In AOP _aspects_ define those points in system, which can should be changed in cohesive way. Such a specification is provided in a declarative or programmatic way.
  * Software systems should be designed with the "naive simple" architecture and deliver some working software to the users. More functionality can be added later on when you're scaling up. (_Doesn't it sound like a MVP?_)

  > Optimal architecture consists of modular problem subdomains; each of them is implemented with use of POJOs. Separate domains are integrated by using non-intrusive aspect-oriented tools. That kind of architecture can be tested similarly to business logic code.

  * Delaying decisions about architecture is good - you can make the decision with most actual information.
  * **Standards are not everything**. Because setting a standard takes a lot of time, usually standards loose the focus on needs of users, who were going to use them.
  * Use **domain specific languages**
  * Regardless whether you design a big system or an individual module - use **simplest, most effective tools.**

