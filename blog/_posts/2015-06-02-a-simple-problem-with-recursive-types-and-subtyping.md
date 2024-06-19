---
title: "A simple problem with recursive types and subtyping"
date: "2015-06-02"
tags: 
  - "infernu"
---

Here's a simple example of recursive types interacting badly with subtyping: `T={ foo: T -> A} U={ foo: U -> B}` Consider `T <: U`, therefore `(T -> A) <: (U -> B)` Which implies: `U <: T` So `T <: U` but also `U <: T`, which is true iff `A <: B` and `B <: A`.

In my case, the subtyping relation is polymorphic subsumption: `T` is subsumed by `U` iff `U` is "more polymorphic", intuitively, it can be instantiated to all types that `T` can.

This situation arises in rather simple code, involving polymorphic vs. non-polymorphic row fields. For example, `A` is a row with a polymorphic method, whereas `B` is a row with a monomorphic (but compatible) method, such as: `A = { method: forall a. a -> () } B = { method: String -> () }` In this case subsumption (the form of subtyping in play) fails.

One way around this is to avoid subsumption issues altogether by keeping things rank-1, and not using higher-rank row fields. Unfortunately, throwing away polymorphic methods is very bad: consider a non-polymorphic `array.map` (in JS).

A slightly better workaround is to push the foralls all the way out, keeping all types (including row types) rank-1. Every time an object method is accessed via the property syntax `obj.method`, we end up instantiating the object's row type, and get a "fresh" type for the method. We get practically polymorphic methods. That's the approach I'm investigating for **[infernu](https://github.com/sinelaw/infernu)**.
