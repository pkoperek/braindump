---
layout: post
title: jUnit creates new instance of test class for each test method
---

This seems to be a bit counter-intuitive. You can use `setUp()` and `tearDown()` methods to setup the test code. On the other hand - the test code can be a bit more concise. Consider those two fragments of code:

```java
class SomeClassTest {

    private int veryImportantField;
    private long anotherVeryImportantField;
    
    @Before
    public void setUp() {
        veryImportantField = 10;
        anotherVeryImportantFieldi = 100L; 
    }

}

```

and

```java
class SimplifiedTest {
    private int veryImportantField = 10;
    private long anotherVeryImportantField = 100L;
}
```

The latter is definitely cleaner and simpler to understand.

I have been using that trick for a while - but I've never found any actual evidence whether this is/isn't a bug. Until today! While reading Martin Fowler's blog I found [this][1] particular post. This was actually a design decision to make it this way. Long live short tests! :)

[1](http://martinfowler.com/bliki/JunitNewInstance.html)
