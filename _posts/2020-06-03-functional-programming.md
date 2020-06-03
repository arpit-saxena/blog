---
layout: post
title: Functional Programming
---

As I was writing down my first post, I realised I was using several functional programming concepts which required some explaining. I started writing the background info in a section then realised it was getting way too big, so I decided to post it here.

This is mostly a crash course of sorts into functional programming explaining the concepts I'll use in upcoming post(s). So here we go:

(Pure) Functional programming works with functions that have no side effects. Best way to understand that is thinking of them as mathematical functions, just mapping a set of values to another set of values. A good technique to see whether a function has no side effect or not is to see whether it can be memoized, i.e. if you store its result, will you get the same result every time you call it with the same parameters?

The advantage of having functions with no side effect is that they make it easier to reason about them. There are also other benefits to it such as the ability to memoize results or use the functions from different parallel processes without worrying about data races.

### Type System

In statically typed functional programming languages (such as ML and Haskell) there is a type system which contains types such as Int, String, etc. This system can be extended by one's own type bindings. A constructor is a function which takes in some parameters and returns a value of the new type, say `m`. There's a special thing about constructors that given a value of type `m`, you can figure out what constructor was used to construct it. So, a definition for `Bool` could have 2 constructors `True` and `False` each taking no argument and returning a value of type `Bool`. A way to think about these types is that the value will have a tag indicating which constructor was used, and other data which would be tha parameters supplied to the function.

### Pattern Matching

We can take information out of the user defined type by a thing called pattern matching. Say we have a type `A` made by two constructors:

```scala
C1: String -> A
C2: Int * Int -> A
```

Given a value `a` of type `A` we can decompose it (so to speak) as:

```scala
a match {
    case C1(s) => ...
    case C2(m, n) => ...
}
```

In the above snippet, the first case would match against a type that's constructed using the constructor `C1` and in the code following `=>`, `s` would be bound to the string that supplied when constructing. Similarly for the constructor `C2`.

### Footnotes

I'm using some Scala syntax in places since I find it to be easier to understand than the concise and terse syntax of Haskell and ML.
