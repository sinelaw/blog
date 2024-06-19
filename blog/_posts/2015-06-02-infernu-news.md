---
title: "infernu news"
date: "2015-06-02"
tags: 
  - "infernu"
---

In the past month, most of the work on [infernu](https://github.com/sinelaw/infernu) was to stabilize the type system, making decisions about some trade-offs. Here's a short summary:

- **Colors!** I refactored all the pretty printing code to use the ansi-wl-pprint package, which fixes indentation and formatting issues and also adds ansi colors to the output. One neat thing is that identical type variables have the same color: [![infernu-colors](images/infernu-colors.png)](https://noamlewis.wordpress.com/wp-content/uploads/2015/06/infernu-colors.png)
- Changed the way constructor functions are treated. Until now, constructor functions were treated at definition site as regular functions. The difference between constructors and non-constructors only happened at "new" calls, where the expected "this" value was forced to be a closed row type. Unfortunately this breaks if you call "new Foo()" inside the definition of "Foo". To avoid this issue, functions with uppercase names are now treated specially and the "this" row-type is forced to be closed when the function name is added to the environment. This allows maximum flexibility while defining Foo, while ensuring Foo's type is closed outside the constructor to prevent junk code like `var x = new Foo(); x.myWrongProperty = 2;`
- Explored the idea of "Maybe" (or "optional") types, including having common JS APIs use them for stronger safety. For example, array access should return a Maybe value. Unfortunately, since there is no syntax to construct a Maybe-typed value (no "Just"), all values can be implicitly "upgradeed" to Maybe. In other words, there is an ambiguity that break type inference. So for now, not implementing Maybe types (perhaps with explicit annotations they can come back).
- Decided to disable generalization of row fields (object properties). This decision means that user-defined objects will by default **not have polymorphic methods**, although the object itself could be polymorphic (and thus different occurrences of the object in the program syntax, will allow instantiation to different types). The reason for this decision is that overly-polymorphic fields cause unexpected type errors, such as when passing objects that contain them as parameters to functions (contravariance can surprise you).
- Started a document sketching out denotational semantics of JS, **as infernu decides these should be**, which helped clarify a few issues in the JS -> LC translator. The next step is to change all translations to preserve semantics, currently they only preserve types.
- Bug fixes: polymorphic subsumption checking, unification of recursive types.
- Increased compatibility: now using base-compat and a custom prelude to increase compatibility with different GHC versions (thanks to RyanGlScott for submitting a fix to use base-orphans which prompted me to do this).
