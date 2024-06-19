---
title: "Inference of 'new' statements, and ambiguous types"
date: "2015-02-09"
tags: 
  - "haskell"
  - "javascript"
---

To prevent possible name clashes, the name of my type inference engine for JavaScript is now [Inferny **Infernu**](https://github.com/sinelaw/inferny) . I hope I don't have to change the name again!

In other news, here is some recent progress:

### 'new' now requires 'this' to be a row type

I changed inference of 'new' calls, so that the constructed function must have a row type as it's "this" implicit parameter (nothing else makes sense).

The change to "new" typing made it possible to define the built in String, Number, Boolean type coercion functions in a way that disallows constructing them (e.g. "new String(3)") which is considered bad practice in JavaScript. The reason is that the constructed values are "boxed" (kind of) so that they don't equate by reference as normal strings, booleans and numbers do. For example, `new String(3) == '3'` but at the same time, `new String(3) !== '3'.`

### Ambiguous Types

I added an initial implementation of what I call ambiguous types. These are simple type constraints that limit a type variable to a set of specific types.

The motivation for type constraints is that some JavaScript operators make sense for certain types, but not all types. So having a fully polymorphic type variable would be too weak. For example, the **+** operator has weird outputs when using all sorts of different input types (NaNNaNNaNNaNNaNNaN....batman!). I would like to constrain + to work only between strings or between numbers.

With the new type constraints, + has the following type:

```
a = (TNumber | TString) => ((a, a) -> a)
```

The syntax is reminiscent of Haskell's type classes, and means: given a type variable "a" that is either a `TNumber` or a `TString`, the type of + is: `(a, a) -> a`

I'm thinking about implementing full-blown type classes, or alternatively, a more powerful form of ambiguous types, to deal with some other more complicated examples.
