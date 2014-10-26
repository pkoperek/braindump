---
layout: post
title: Notes on "Clean Code - Boundaries" chapter
---

  * Encapsulate foreign/alien APIs (create own interfaces and adapters using the foreign API).
  * Create _"learning"_ unit tests - tests which verify that your usage of API is correct. 
    _"Learning"_ - because you actually learn the API by writing them. Such tests can be later 
    used eg. to ensure that new version of framework/library behaves accordingly (that they are 
    compatible with the way you use them).
