---
layout: post
title: Provided scope in Gradle
comments: true
---

Very useful thing (especially when you're more familiar with maven :)): [link](http://forums.gradle.org/gradle/topics/how_do_i_best_define_dependencies_as_provided).

The trick is to create a `provided` configuration and add it's artifacts to compile. 

**EDIT:** You often need to add the `provided` scope separately in idea plugin configuration: [this](http://www.gradle.org/docs/current/dsl/org.gradle.plugins.ide.idea.model.IdeaModule.html) explains how.
