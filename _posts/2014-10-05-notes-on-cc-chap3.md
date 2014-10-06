---
layout: post
title: notes on "Clean Code - Functions" chapter
---

Main thoughts I want to remember from that chapter:

  * Functions should be **small**. **Small** means up to several lines.
  * Functions should have **descriptive names**.
  * Functions should **do one thing**. They should **do it well**. They should **do it only**. THis means that the side effects should be as limited as possible (preferably - there should be none).
  * It should be possible to read the function as a "To ..." sentence:
    
    ```  
    TO functionName we do something and then we do something else...
    ```

  * Functions should operate on a **single level of abstraction**.
  * **switch** statements should be avoided. They can probably get replaced by inheritance - if not should be hidden in a single place in such way that it won't be necessary to create similar *switch* statements again.
  * The smaller the number of arguments == the better. Rule of thumb: **three parameters is almost always too much**. One and two parametrs is acceptable.
  * Don't use arguments as a way to communicate with outer world - don't use output parameters.
  * Throw exceptions instead of returning error codes. You don't enforce handling the error right away - it can be dealt with in a convenient place in code.

**Remember: functions are not written correctly at first. Write something meeting functional requirements with TDD and then refactor mercilessly.**
