---
layout: post
title: "any" and "isA" matchers in _mockito_
comments: true
---

The most obvious interpretation of `any()` matcher in mockito is that it accepts _any_ object. Ok - and what about `any(Clazz.class)`? It accepts any object of class `Clazz`. 

Actually those two sentences contain fallacies... First: `any` accepts also null parameters (it doesn't require the argument to actually exist!). Secondly: `any(Clazz.class)` accepts any object of ... any class! Actually, in most cases it would be probably better to use a `isA` matcher - which simply checks the class of passed object (the object needs to exist in the first place!). 
