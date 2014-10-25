---
layout: post
title: Notes on "Clean Code - Error Handling"
---

  * _"Error handling is important, but if it obscures logic, it's wrong"_
  * Using exceptions make it possible to decouple business logic from error handling.
  * Using unchecked exceptions in Java is _preferable_. Checked exceptions violate Open/Closed principle. Example:

```java
class A {
    void methodA() {}
}

class B {
    void methodB() {
        new A().methodA();
    }
}

class C {
    public static void main(String[] args]) {
        new B().methodB();
    }
}
```

  If I want to add checked exception to `methodA()` I need to change signature iun `methodB()` - which shouldn't care about it at all.
  *Encapsulation is broken!*

  * Provide context to exception which explains why the operation failed. 
  * **Sometimes it is required to handle couple different exception types coming from a single method call in the same way. 
    In such case, it might be better to create a wrapper class, which will encapsulate that call and throw a single, common 
    exception type (containing the original cause). This way the original invocation will need to handle only a single 
    exception - this will reduce duplication.**
  * Don't return _null_ values - return empty/guard values (_Special Case Pattern_).
  * Don't pass null values as arguments.
