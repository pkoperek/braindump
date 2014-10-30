---
layout: post
title: Provided scope in Gradle
comments: true
---

Very useful thing (especially when you're more familiar with maven :)): [link](http://forums.gradle.org/gradle/topics/how_do_i_best_define_dependencies_as_provided).

The trick is to create a `provided` configuration and add it's artifacts to compile. 
