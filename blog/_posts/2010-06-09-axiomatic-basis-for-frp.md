---
title: "Axiomatic basis for FRP"
date: "2010-06-09"
categories: 
  - "frp"
---

# What's this about?

I've been thinking about ways to lay firm, logical foundations for a model of FRP. Rather than trying to cook up the "next best thing", we should perhaps review what we really want and try to infer what we need. Unfortunately I've very little time to think about this and I've hardly discussed this. Hopefully this blog post will give my effort another small push.

# Goal

The idea is to define a set of building blocks that allow one to express only systems that satisfy a few basic properties. Specifically we want systems that _don't_ satisfy these properties, to be inexpressible within our model. It makes sense to start by defining the properties we want to satisfy, and then to try and deduce the most general model that satisfies them and only them.

The basic concept is that of a "system", which we define as a function:

`f :: (T -> A) -> (T -> B)`

Where _T_ is an ordered, measurable set, usually denoting time. This is what Conal Elliott called an "[interactive behavior](http://conal.net/blog/posts/why-classic-FRP-does-not-fit-interactive-behavior/ "interactive behavior")", and what is known in Yampa as a "signal function". I use the term "system" because that's how this is called in engineering, and I see no reason to invent new names.

# Desired properties of systems

**Edit: the properties as defined below are not _exactly_ what we want. See comments for discussion.**

So, here are the requirements I'd like to propose. Every expressible system must satisfy the following:

1. **Insensitivity:** If two inputs are almost the same (differ at most at a set of measure zero), the output will be _(almost?)_ the same.
2. **Causality**: For all t in T, output(t) can only depend on input(t-s) where _s ≥ 0_. Meaning, for every two inputs x1, x2 that have the same past, the output for the present is the same. And to be more precise, for every _t_, if _x1(t-s) = x2(t-s)_ for almost all _s ≥ 0_, the output at _t_ will be the same.
3. **Markovity:** Same as (2), but with the condition that _|s| < ∞_. In words, the output does not depend on inputs from the infinitely remote past or future.
4. **Finite memory:** given the current input (a single value), the current output depends additionally at most on a finite-length binary string (that may have been computed from past inputs).
5. **Time invariance** (_optional_): If _x(t)_ produces output _y(t)_, then _x(t - s)_ produces output _y(t - s)_.

## Discussion

The first property means that the system is not overly sensitive. Integration is one example of a system that satisfies this property: changing the value of a function at finite number of points does not affect the value of its integral. I don't know how to call this - if anyone knows a name for this property, please tell me. For now I'll call it _insensitivity_.

The second property is _causality._ It stems from the physical intuition that we can't know the future, and every value that we  (or our systems, or any other natural thing) compute at the present, should depend only on inputs from the past (or at most, the present). Otherwise, we'd have to wait until the future "arrives" to complete the computation.

Causality is one of the properties that are easily violated if we consider a model that allows us to arbitrarily sample our input, one that allows us to actually treat the input as a function of time. Mathematically, a function is just a mapping and has no rules regarding which part of the domain can be evaluated and when. So, the first realization from our requirements is that we can't  have signals (inputs, outputs, etc.) as directly accessible values in our model, or that they shouldn't be modeled as functions. Yampa (or in it's other name, AFRP) imposes the no-access rule and doesn't have signals as values that can be passed around. Instead, they are indirectly manipulated by constructing systems that transform them - a sort of "wiring up". There's a big debate on the pros and cons of an Arrow-based FRP, but I've yet to understand the full consequences.

As for a non-functional description of signals, I haven't though about it enough to know if there's any obviously useful alternative that solves the causality problem. So in my case, I choose to go the Yampa way - I won't provide any means to access signals directly.

Besides causality, we also want to make sure the system does not depend on the infinitely remote past. Physical intuition says that we'll never be able to know what happened that far back. This is the defining property of Markov processes (of any order), so I'll call this _Markovity_. Note that the requirement is defined as going both ways, but our previous demand for causality means that it only has meaning regarding past inputs. Perhaps the more general formulation (that includes both past and future) is redundant, but to me it seems nicer to have it symmetric even though only one direction (past) matters in our case.

The fourth property is _finite memory_, which is physically intuitive. Especially in the context of computers, we don't want our systems to endlessly accumulate information until we run out of memory. Mathematically, this requirement needs elaboration to be made precise. The formulation I've given above (depending on a finite-length binary string) is simple and describes what I want, but it will require extra work to figure out what implications it has on our systems.

Finally, _time invariance_ can be interpreted as not being able to tell the absolute "wall clock" time. Physically this is true: we don't know about any absolute time. Although in reality engineered systems do in fact evolve over time (wear and tear, for one) it is not some intrinsic property of the system that causes this change - rather it is that the entire observable universe serves as inputs to our systems, which can never be fully isolated. Thus the only thing that should matter for our system's output is how the input evolves over time, not the exact position in global time of the input. We forbid sensitivity to input time shifts. I noted this requirement as optional because I'm not sure we really want it as a limitation. I do have a gut feeling that we don't miss out any desirable systems by adding this limitation - please correct me if you have any ideas against this.

# What's next?

In this blog post I've identified the five basic properties that we desire in a system. The next step should be to define a set of operations that can serve as universal building blocks for any system that satisfies the requirements. Perhaps more importantly, our building blocks should _prevent_ us from accidentally expressing systems that violate the requirements. I'll even say that it's ok if a few valid systems are inexpressible (our "systems filter" has false positives) as long as no violating system is expressible (no false negatives).

I'm waiting for your comments.
