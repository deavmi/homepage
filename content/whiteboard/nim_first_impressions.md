---
title: First impressions of Nim
author: Tristan B. Kildaire
date: 2022-12-28
---

Damn Nim's type system is quite nice

I am getting the hang of it

Already wrote some nice compile-time code

No macro stuff yet though but havent needed that just yet

That can be useful for certain things

Nim makes a distinction between its meta programming facilities

the generics/templates and macros

both are meta programming

but the latter focuses solely on AST transformations and provides a whole system for that
In D's world you simply use one system for manipulating alles, both for meta programming constraints and for yeah AST stuff

Although I must say

D's ast manipulation ain't all that much tbh

Nim's seems to let you actually twiddle aspects of body and functions and return values

In D you don't get direct conteol over that

You could however do it via generics

or if you want actual direct control then you can use mixins and append strings together, it is messy but can be done

I did that for my Gogga logging library to build a DEBUG, INFO, etc function

By compile-time iterating over the LogLevel enum members and mixin(string)-ing a new defintion per each

---

Disclaimer: The first time I **ever** looked at Nim was back when I was in highschool.
Suffice it to say I understand the language a lot better now.
