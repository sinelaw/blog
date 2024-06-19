---
title: "Ad-hoc polymorphism, type families, ambiguous types, and Javascript"
date: "2015-02-19"
tags: 
  - "haskell"
  - "infernu"
  - "javascript"
---

Imagine a language, where all we have are strings and numbers, and where + is a built-in function that can either add numbers or concatenate strings. Consider the following code:

```
a + b

```

What are the types of `a`, `b` and `+`?

Using a Haskell-like type signature we can say that + has either of these types: `+ :: (Number, Number) -> Number` or `+ :: (String, String) -> String` (Currying avoided intentionally.)

This is a classic case of ad-hoc polymorphism. With type classes one could say:

```
class Plus a where
  (+) :: (a, a) -> a

instance Plus Number where
  x + y = ... implement addition ...

instance Plus String where
  x + y = ... implement string concatenation ...

```

That's great! Now we can type our `a + b`:

```
a + b :: Plus t => t

```

Where `a :: t` and `b :: t`.

Yes, there are also Scala-style implicits ([recently discussed here](http://www.reddit.com/r/ocaml/comments/2vyk10/modular_implicits/) in a proposal for OCaml), and probably other solutions I'm less aware of.

Notice that in the + example, a constraint on a **single type** (expressed through type class requirements) was enough to solve the problem.

### Constraints on multiple types

Now, let's look at a more complicated problem, involving multiple types at once.

Consider a language with only parameterized lists `[a]` and maps from strings to some type, `Map a`. Now, throw in a terribly ambiguous syntax where brackets denote either accessing a list at some index, or accessing a map at some key (_wat_):

```
   x[i]

```

That syntax means "access the container x at index/key i". Now, what are the types of `x`, `i` and the brackets? There are two possibilities: if `x` is an array, then `i` must be a number; otherwise if `x` is a map, then `i` is a string.

[Type families](https://wiki.haskell.org/GHC/Type_families) can be used to encode the above constraints:

```
class Indexable a where
  type Index a
  type Result a
  atIndex :: (a, Index a) -> Result a

```

The syntax means that any instance of the type class `Indexable a` "comes with" two accompanying types, `Index a` and `Result a` which are uniquely determined by the appropriate choice of `a`. For \[t\], Index = Number and Result = t. For Map t, Index = String and Result = t.

Now we just need syntax sugar where ``x[i] = x `atIndex` i``. We could then define instances for our two types, \[a\] and Map a (remember, in our language the map keys are always Strings):

```
instance Indexable [a] where
  type Index [a] = Number
  type Result [a] = a
  atIndex = ... implement list lookup by index ...

instance Indexable (Map a) where
  type Index (Map String a) = String
  type Result (Map String a) = a
  atIndex = ... implement map lookup by key ...

```

Nice. Now, to type our original expression `x[i]`:

```
x[i] :: Indexable a => Result a

```

Where `x :: a` and `i :: Index a`.

Great! Type families (or rather, "type dependencies") provide a way to represent inter-type constraints and can be used to resolve ambiguous expressions during type inference. (I've heard that type families are equivalent to functional dependencies or even GADTs for some value of "equivalent" , maybe where "equivalent = not equivalent at all", but that's off-topic.) See also [a valid Haskell implementation of the above example](https://gist.github.com/sinelaw/a4a035a5bb6b0d6d6c1c) (thanks to Eyal Lotem).

I don't know if Scala-style implicits can be used to such effect - let me know if you do.

### Ambiguous Types

Now, here's an altogether different way to approach the ad-hoc polymorphism problem. This was my idea for a solution to ad-hoc polymorphism with inter-type constraints, before I realized type families could do that too.

Define an "ambiguous type" as a type that represents a disjunction between a set of known types. For example, for + we could say:

```
+ :: (a = (String | Number)) => (a, a) -> a

```

The type variable `a` is constrained to be either a String or a Number, explicitly, without a type class. I see two main differences with type classes, from a **usage** point of view. First, this type is closed: because there is now class, you can't define new instances. It must be a Number or a String. Second, you don't need to add + to some class, and if we have more operators that require a similar constiant, or even user-defined functions that require some kind of ad-hoc constraint, we don't need to define a type class and add functions/operators to it. Lastly, the type is straightforward, and is easier to explain to people familiar with types but not with type classes.

By the way, I have no idea (yet) if this leads to a sound type system.

#### Ambiguous types involving multiple type variables

Let's continue to our more complicated, ambiguous "atIndex" function that can either index into lists (using a number) or into maps (using a string). A simple disjunction (or, |) is not enough: we must express the dependencies between the container type and the index type. To that end we add conjunctions (and, &) so we can express arbitrary predicates on types, such as `(t & s) | u`. The type of atIndex will now be:

```
atIndex :: (a = [t] & i = Number) | (a = Map t & i = String) => a -> i -> t

```

This definitely does the job. Is it sound? Will it blend? I don't know (yet).

The main drawback of this system is the combinatorial explosion of the predicate when combining ambiguous (overloaded) functions, such as in the following program:

```
x[i] + y[j]

```

`x` could be either a list or map of either numbers or strings, and so can `y`, so `i` and `j` can be either numbers or strings (to index into the lists or maps). We quickly get to a very large predicate expression, that is slow to analyze and more importantly, very hard to read and understand.

Nevertheless, I implemented it.

### Implementing ambiguous types

[Infernu](https://github.com/sinelaw/infernu) is a type inference engine for JavaScript. All of the examples described above are "features" of the JavaScript language. One of the goals of Infernu is to infer the safest most general type. JS expressions such as `x[i] + d` should be allowed to be used in a way that makes sense. To preserve safety, Infernu doesn't allow implicit coercions anywhere, including in +, or when indexing into a map (JS objects can act as string-keyed maps). To retain a pseudo-dynamic behavior safely, polymorphism with fancy type constraints as discussed above are required.

To properly implement ambiguous types I had to solve a bunch of implementation problems, such as:

1. How to unify two types that have constraints on them? The problem is of finding the intersection of two arbitrary boolean expressions. My solution was to convert both equations to [DNF](http://en.wikipedia.org/wiki/Disjunctive_normal_form), which is just a big top-level OR between many AND (conjunction) expressions (and no further OR or ANDs). Then compare every combination pair of conjunctions and rule out ones that don't match. The surviving set of conjunctions is used to build a new DNF expression.
2. How to simplify the resulting predicate? While an [optimal solution](http://en.wikipedia.org/wiki/Quine%E2%80%93McCluskey_algorithm) exists, it is NP-hard, and more importantly, I was too lazy to implement it. Instead I ended up using a heuristic constructive approach where while building expressions, I try to find patterns that can be simplified.
3. A software engineering challenge: when building the constraint unifier, how to design an API that allows effectful unification at the leaf nodes when we are testing conjunctions? I ended up using some Applicative/Traversable magic (but the code isn't pretty). Rumor has it that lenses may make it much nicer.
4. How to represent constraints in the types AST? I followed what is allegedly GHC's approach by defining a wrapper data type, which wraps a type with an accompanying predicate, something like: `data QualType t = QualType (Pred t) (Type t)`. Then, the unification function is left unchanged: when doing inference, I also call the constraint unifier separately, using its result to "annotate" the inferred type with a constraint predicate.

## Conclusions

Apparently, the ambiguous types are not a good solution due to the complex type signatures. I'll either leave the ambiguous types in (having already implemented them) or just get rid of them and implement type families, which will require another crusade on my code.

I'm still thinking about whether or not type families cover all cases that my ambiguous types can. One example is the "type guard" pattern common in JavaScript:

```
if (typeof obj == 'string') {
    .. here obj should be treated as a string! ...
}

```

Can ambiguous types and type families both be used coherently to implement compile-type checking inside type guards? (Haven't given it much thought - so far I think yes for both.)

Also, since ambiguous types are closed, they may offer some features that type families can't, such as warnings about invalid or incomplete guards, which can be seen as type pattern matching. Maybe closed type families are a solution: I don't know much about them yet.

I also don't know if ambiguous types lead to a sound type system or are there pitfalls I haven't considered. Remember that these ambiguous types may also interact with many features: parameteric polymorphism, row-type polymorphism, the requirement for prinicpal types and full type inference without annotations, etc.

Lastly, I've probably re-invented the wheel and somebody has written a bunch of papers in 1932, and there's some well-accepted wisdom I've somehow missed.

### Acknowledgement

Thanks to Eyal Lotem for a short, but very fruitful conversation where he showed me how type families may obsolete my idea.
