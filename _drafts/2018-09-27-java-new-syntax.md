---
layout: post
title: Modern Java Review - New Syntax Features
comments: true
---

## Default methods in interfaces

* Prior to Java 8, interfaces contained only `public static final` variables
  (actually constants) and `public abstract` methods.
* Right now there might be actually some implementation.
* `default` methods are public by default, access modifier is optional though.
* They _must_ have a body.
* `default` keyword can be _only_ used in context of interface method
  signatures, not in class method signatures.

## Static methods in interfaces

* Starting with Java 8, interfaces can include static methods with concrete
  implementations.
* Those methods are public by default (can't be private, protected or final).


Source: 
