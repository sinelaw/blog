---
title: "Two implementations of DHM type inference"
date: "2015-09-24"
tags: 
  - "haskell"
---

Here are two simple implementations of Damas-Hindley-Milner type inference.

First, is [my Haskell version of a region-based optimized type checker](https://github.com/sinelaw/fresh/blob/levels-lazy/Main.hs) as explained by Oleg Kiselyov, in his [excellent review of the optimizations to generalization used in OCaml](http://okmij.org/ftp/ML/generalization.html). Oleg gives [an SML implementation](http://okmij.org/ftp/ML/generalization/sound_lazy.ml), which I've Haskellized rather mechanically (using ST instead of mutable references, etc.) The result is a bit ugly, but it does include all the optimizations explained by Oleg above (both lambda-depth / region / level fast generalization and instantiation, plus path compression on linked variables, and not doing expensive occurs checks by delaying them to whenever we traverse the types anyway).

Second, here's my [much shorter and more elegant implementation](https://github.com/sinelaw/fresh/blob/unification-fd/Main.hs) using the neat [unification-fd package](https://hackage.haskell.org/package/unification-fd) by Wren Romano. It's less optimized though - currently I'm not doing regions or other optimizations. I'm not entirely satisfied with how it looks: I'm guessing this isn't how the author of unification-fd intended generalization to be implemented, but it works. Generalization does the expensive lookup of free metavariables in the type environment. Also the instantiate function is a bit clunky. The unification-fd package itself is doing inexpensive occurs checks as above, and path compression, but doesn't provide an easy way to keep track of lambda-depth so I skipped that part. Perhaps the Variable class should include a parameterized payload per variable, which could be used for (among other things) keeping track of lambda-depth.
