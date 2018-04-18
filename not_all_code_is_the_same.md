Not all code is the same
=======

### Fighting the hard way for Clean Architecture and Separation of Concerns

> TL;DR Code might be writing in the same language and might be residing in the same file, yet its type and character might (and will) be different, and that changes everything.

We developers are often hypnotized by our code: we pick a primary language for a project, we write stuff in it. We use language features (say, list comprehensions) so they become quite evenly distributed across source files. That make code base quite coherent and homogenous. Which makes us think (subconsciously) that one piece is like another.

To my perspective, this is the major reason why we so fail so miserably at topic of Separation of Concerns. Code looks even: no more concerns to extract!

But this is a mirage. If we look closer, code is very different, it behaves differently, each type has its own lifecycle, etc. (see aspect oriented programming).

We need to extract each type and make it easy for every one to evolve independently. This will save us enormous time in maintenance and will help safely grow our application in size.

So which types of code can be found?
(Remember, nothing really new here, just paraphrasing and summarizing good old books)

- code for object construction and for object Usage
- code to define view layouts (structure)
- code to define view styling
- code which ties application to platform's framework (e.g. Activity lifecycle in Android)
- code which drives a particular screen (presentation logic) vs. code which retrieves and changes state of application
- code which describes DTOs (API entitites) vs domain objects
