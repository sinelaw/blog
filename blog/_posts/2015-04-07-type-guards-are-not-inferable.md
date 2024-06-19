---
title: "Type guards are not inferable"
date: "2015-04-07"
---

Type guards, or typecase expressions allow refining a binding's type and are commonly used in dynamic languages such as JavaScript. Consider (adapted from [Simple Unification-based Type Inference for GADTs](http://research.microsoft.com/en-us/um/people/simonpj/papers/gadt/gadt-icfp.pdf "Simple Unification-based Type Inference for GADTs")):

```
f x y = typecase x of
            Int -> i + y
            other -> 0


```

The function can have either of these two types (among others):

`a -> Int -> Int`

And

`a -> a -> Int`

Neither is more general, so there is no principal type.

Unless the user tells us something (in a type annotation) we have no way of deciding which type should be inferred.

So for now, [Infernu](https://github.com/sinelaw/infernu) will probably not have support for type guards like [Flow](http://flowtype.org/) and [TypeScript](http://www.typescriptlang.org/) do (both of which don't have full type inference). Another possibility is to allow type guards in trivial cases only.
