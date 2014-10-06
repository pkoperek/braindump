---
layout: post
title: notes on Clean Code - Comments chapter
---

  * Better refactor code than comment it.
  * Maintaining code takes time and requires great discipline - what usually is hard in fast-paced projects.
  * Don't rely too much on comments: **they get outdated** very easily. This makes them innacurate, not relevant, misleading or just simply lying.
  * Instead of commenting out code - throw it away. We have a copy of it in code versioning system.
  * "HTML in source code comments is an abomination".
  * Most comments are useless (too far from code, too obvious, just mumbling, not accurate enough).

When comments might be a good idea:

  * Legal obligation to write specific comments (copyrights).
  * Clarification of code which just can't be written in more expressive way (quite rare).
  * Docs on public API.

What I disagree with:

  * TODO comments are bad - not good. Instead of TODO comment there should be a task in backlog, explaining what should be corrected.
