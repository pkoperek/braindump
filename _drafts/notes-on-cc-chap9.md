---
layout: post
title: Notes on "Clean Code - Tests" chapter
comments: true
---

  * Three laws of TDD:
> 1. You may not write production code until you have written a failing unit test.
> 2. You may not write more of a unit test than is sufficient to fail, and not compiling is failing.
> 3. You may not write more production code than is sufficient to pass the currently failing test.

  * How fast is TDD this way? 
    * According to Uncle Bob, you can write over a dozen such tests each day.
    * On the other hand, a loop of writing test, running failing test, writing prod code, running passing test should take around 30 seconds.
  * Quick and dirty tests are as good as no tests at all. They end up as being hard to maintain and get quickly dropped when first problems occur. 
  * **"Test code is as important as production code."** - it needs to be maintained, refactored and requires and requires a lot of thought and attention.
  * **Tests vastly enhance possibilities to work with the code - they enable introducing changes in a safe and controllable manner**
  * Test code has a different set of engineering standards than the production code. It can be less performant - but needs to be simple, readable and cohesive.
  * Ideally there should be a single assertion per test. This is a good advice, however there is a better rule: **test one concept per test case (answer a single question)**. Some questions require more than one assertion to be answered.
  * **Five rules of unit tests: F.I.R.S.T:**
> **Fast** - as fast as possible - ideally couple seconds - we want to run them as often as possible
> 
> **Independent** - each test case should run alone; if a test fails - it fails because of its own specific reason - not because the setup test case failed
>
> **Repeatable** - should run in the same way in every environment
> 
> **Self-validating** - they are automatically validated; test has binary output: passed or failed
>
> **Timely** - written in proper moment - before the production code
