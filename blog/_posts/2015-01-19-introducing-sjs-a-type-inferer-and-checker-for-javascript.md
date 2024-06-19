---
title: "Introducing SJS, a type inferer and checker for JavaScript (written in Haskell)"
date: "2015-01-19"
---

TL;DR: SJS is a type inference and checker for JavaScript, in early development. The core inference engine is working, but various features and support for the full browser JS environment and libraries are in the works.

SJS (Haskell [source on github](http://github.com/sinelaw/sjs)) is an ongoing effort to produce a practical tool for statically verifying JavaScript code. The type system is designed to support a **safe subset of JS**, not a superset of JS. That is, sometimes, otherwise valid JS code will not pass type checking with SJS. The reason for not allowing the full dynamic behavior of JS, is to **guarantee more safety** and (as a bonus) allows fully unambiguous type inference.

The project is still in early development, but the core inference engine is more or less feature complete and the main thing that's missing is support for all of JS's builtin functions / methods and those that are present in a browser environment.

Compare to:

- [Google Closure Compiler](https://developers.google.com/closure/compiler/), whose primary goal is "making JavaScript download and run faster", but also has a pretty complex type-annotation centric type-checking feature. The type system is rather Java-like, with "shallow" or local type inference. Generics are supported at a very basic level. I should write a blog post about the features and limitations of closure. It's a very stable project in production use at Google for several years now. I've used it myself on a few production projects. Written in Java. They seem to be working on a new type inference engine, but I don't know what features it will have.
- [Facebook Flow](https://code.facebook.com/posts/1505962329687926/flow-a-new-static-type-checker-for-javascript/), which was announced a few weeks ago (just as I was putting a finishing touch on my core type checker code!), has a much more advanced (compared to closure) type checker, and seems to be based on data flow analysis. I haven't gotten around to exploring what exactly flow does, but it seems to be much closer in design to SJS, and obviously as a project has many more resources. There are certain differences in the way flow infers types, I'll explore those in the near future.
- [TypeScript](http://www.typescriptlang.org/): a **superset** of JS that translates into plain JS. Being a superset of JS means that it includes all of the awful parts of JS! [I've asked](https://typescript.codeplex.com/discussions/434723) about disabling those bad features a while back (around version 0.9); from what I've checked, version 1.4 still seems to include them.
- Other something-to-JS languages, such as [PureScript,](http://www.purescript.org/) [Roy](http://roy.brianmckenna.org/), [Haste](http://haste-lang.org/), and [GHCJS](https://github.com/ghcjs/ghcjs) (a full Haskell to JS compiler). These all have various advantages. SJS is aimed at being able to run the code you wrote **in plain JS**. There are many cases where this is either desired or required.

Of all existing tools, Flow seems to be the closest to what I aim to achieve with SJS. However, SJS supports type system features such as polymorphism which are not currently supported by Flow. On the other hand, Flow has Facebook behind it, and will surely evolve in the near future.

Closure seems to be designed for adapting an existing JS code base. They include features such as implicit union types and/or a dynamic "any" type, and as far as I know don't infer polymorphic types. The fundamental difference between SJS and some of the alternatives is that I've designed SJS for **more safety**, by supporting a (wide) subset of JS and disallowing certain dynamic typing idioms, such as assigning values of different types to the same variable (in the future this may be relaxed a bit when the types are used in different scopes, but that's a whole other story).

Ok, let's get down to the good stuff:

**Features:**

- Full type inference: no type annotations necessary.
- Parametric polymorphism (aka "generics"), based on Hindley-Milner type inference.
- Row-type polymorphism, otherwise known as "static duck typing".
- Recursive types for true representation of object-oriented methods.
- Correct handling of JS's `this` dynamic scoping rules.

Support for type annotations for specifically constraining or for documentation is planned.

Polymorphism is value restricted, ML-style.

Equi-recursive types are constrained to at least include a row type in the recursion to prevent inference of evil recursive types.

## [](https://github.com/sinelaw/sjs#example)Examples

**Note**: An ongoing goal is to improve readability of type signatures and error messages.

### [](https://github.com/sinelaw/sjs#basic)Basic

JavaScript:

```
var num = 2;
var arrNums = [num, num];
```

SJS infers (for arrNums):

```
[TNumber]
```

That is, an array of numbers.

Objects:

```
var obj = { something: 'hi', value: num };
```

Inferred type:

```
{something: TString, value: TNumber}
```

That is, an object with two properties: 'something', of type string, and 'value' of type number.

### [](https://github.com/sinelaw/sjs#functions-and-this)Functions and `this`

In JS, `this` is one truly awful part. `this` is a dynamically scoped variable that takes on values depending on how the current function was invoked. SJS knows about this (pun intended) and infers types for functions indicating what `this` must be.

For example:

```
function useThisData() {
    return this.data + 3;
}
```

SJS infers:

```
(this: {data: TNumber, ..l} -> TNumber)
```

In words: a function which expects `this` to be an object with at least one property, "data" of type number. It returns a number.

If we call a function that needs `this` incorrectly, SJS will be angry:

```
> useThisData();
Error: Could not unify: {data: TNumber, ..a} with TUndefined
```

Because we called `useThisData` without a preceding object property access (e.g. `obj.useThisData`), it will get `undefined` for `this`. SJS is telling us that our expected type for `this` is not unifiable with the type `undefined`.

### [](https://github.com/sinelaw/sjs#polymorphism)Polymorphism

Given the following function:

```
function makeData(x) {
    return {data: x};
}
```

SJS infer the following type:

```
((this: a, b) -> {data: b})
```

In words: A function that takes anything for its `this`, and an argument of any type, call it `b`. It returns an object containing a single field, `data` of the same type `b` as the argument.

### [](https://github.com/sinelaw/sjs#row-type-polymorphism-static-duck-typing)Row-type polymorphism (static duck typing)

Given the following function:

```
function getData(obj) {
    return obj.data;
}
```

SJS infers:

```
((this: h, {data: i, ..j}) -> i)
```

In words: a function taking any type for `this`, and a parameter that contains **at least one property**, named "data" that has some type `i` (could be any type). The function returns the same type `i` as the data property.

SJS is an ongoing project - I hope to blog about specific implementation concerns or type system features soon.
