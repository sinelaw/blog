---
title: "Axioms for FRP: Discussing insensitivity"
date: "2010-06-13"
categories: 
  - "frp"
---

[Last post](http://noamlewis.wordpress.com/2010/06/10/axiomatic-basis-for-frp/ "Axiomatic basis for FRP") I proposed a set of (five) properties that every system should have. In following posts I'll try to refine these properties and perhaps change, add or discard some of them. The comments I got really made me think, so thanks for commenting (keep it up!).

## Insensitivity

The first property I defined, was given the made-up name of "insensitivity":

> If two inputs are almost the same... the output will be _(almost?)_ the same.

Here are a few points about it. To make things clearer, let's define "almost everywhere" as "everywhere except a set of measure zero".

The reason for not using only continuity conditions on systems was to avoid the various pitfalls encountered by mathematicians as they developed integration and analysis. I'm no mathematician (not even an undergraduate one) but I do know that [Lebesgue integration](http://en.wikipedia.org/wiki/Lebesgue_integration) is considered the best (even final) solution to those various problems, and I'm trying to base my systems on this result. A good example is that of the characteristic function of the rationals (also known as [Dirichlet's function](http://mathworld.wolfram.com/DirichletFunction.html)) which equals 1 for rationals and 0 for all other reals. Traditional integration is undefined on such functions - which means that integrals are not total functions. With Lebesgue integration, the issue is solved.

On the other hand, we _do_ want our systems to "behave nicely" and not wildly change their outputs with every small change of input. I'm not talking about changes of the input signal in time - rather, I mean that if we replace the entire input signal with one that is very similar, the output should be similar. I'm still not sure how to cure this problem completely, but the way I defined "insensitivity" ensures that at least for inputs that differ at a _very_ small ("almost nowhere") subset of time, the output will be the same. I'm not sure, but we may still want to require systems to be continuous (changes in input that are _more_ than "almost nowhere" cause small changes in the output).

This last point brings up the following question, raised by both Derek Elkins and Luke in the comments on that last post. Citing Derek:

> ...without the “almost” on the output you would disallow the identity “system” which I highly doubt you want to do.

The identity system is a good way to bring out the meaning of insensitivity. I do think that the guiding principle should be that "almost identical" _signals_ are considered identical, and in the same spirit "almost identical" _systems_ are also identical. Thus if two systems are the identity system almost everywhere, except for a zero-measure set of different values between the systems, they are still considered the same system. My opinion is that it's better to not require the existence of a single identity system. Alternatively, we define that every two systems that produce the same output almost everywhere for all inputs, are "equal".

Note I've added the "(almost?)" on the insensitivity requirement on the output (see above). I think I still want to keep that "almost" on the output. The reasoning is twofold. First, for consistency we should make no distinction between output signals that are the same almost everywhere. Secondly, I don't want to _force_ systems to give exactly the same output when inputs are not exactly the same (not just "almost everywhere").  I'm not entirely certain about this, though.

In any case I don't want the "almost" on the output be used as way to process events. Since we consider "almost identical" systems to be equal, such usage is pointless anyway. So what do we do with events?

### Is this an implementation thing?

No, I'm not requiring insensitivity because the implementation may cause "glitches" in the output (if I understand correctly that was one of Derek's questions). In fact I'm not considering implementation at all at this point. I'm trying to come up with the basic properties that we would require of systems in an ideal setting.

### Wait! What about events?

Almost all the comments raised this point. Truth is I sort of ignored events when I wrote that post, but I did originally have them in mind (and then forgot about them completely). Since we ignore differences that occur on zero-measured subsets of the input, there is indeed no way to "detect" or differentiate between inputs that contain single-point values. So how do we handle events, or values that occur at single points in time?

Recall that we defined the time domain as an arbitrary ordered, measurable set. Here is where the generality of that definition comes in. If we ignore the idea of signals combining events and continuous values, I think we can solve this. We can try to handle events by defining an appropriate time domain, and allowing systems' outputs and inputs have a different time domain (unlike the definition I gave in the previous post).

#### An attempt at defining event signals

The following is an attempted definition for event signals. Note that this attempt does not allow simultaneous events (which can be worked around by having compound types for the event values) or combined discrete/continuous time signals. Perhaps these two issues demand direct resolution, but to make progress in the current direction I'm ignoring them.

Let T (the time domain) be a countable set of positively increasing reals signifying event times. It is easy to define an ordering between disjoint interval-type subsets, based on the natural ordering of the reals. For a measure, we can count the number of elements in a subset (the [counting measure](http://en.wikipedia.org/wiki/Counting_measure)). A single event (a subset containing one time element) would have measure 1, two events - measure 2, etc.

Thus, an event signal is a function T -> A that maps every element of the time set to an event value. Notice how the T here is necessarily different for each event signal (the times contained in the set are different). I'm not sure it's bad - we can still define comparisons between event signals. The insensitivity condition in this case, is still valid: T is measurable and every non-empty subset in T has non-zero measure, so no events get "missed" by the insensitivity.

So to summarize, we can handle events by considering a different time domain (a discrete one, essentially) and use an appropriate measure on that domain. It makes sense to allow systems to have outputs with different time-types than the input - for example a "sampling system" takes real time in the input, and outputs in a discrete time. Remember, I'm not talking about how to implement or represent this in Haskell (or otherwise) - I'm just trying to define the desired model in an ideal setting.

#### Delta functions

Some (including myself) have suggested to use something akin to [Dirac's delta function](http://en.wikipedia.org/wiki/Dirac%27s_delta_function) (which isn't a function, but a measure or a distribution) to represent events on real, continuous time. Adequate discussion is too long for this post. My guess is that since these "mathematical constructs" are not functions of time at all, and since they are originally used as measures, we will more or less need to re-invent them for our wider context.

One way to define the delta is given here. The _δ_ function is an operator that takes a "test function" _f_ of time, and then for any given time interval (or more generally, a subset of time) has one of two values. So <δ,f> on U where U is a subset of R, is:

- If U contains the origin (0 is zero in this font), the value is _f(0)_.
- Otherwise, the value is 0.

The delta can be shifted to obtain the same on points other than the origin. Essentially, the delta function tells us whether the given subset of time contains a particular point, and then evaluates to either a default value (0) or the value of the given function at that point. The sum of two non-overlapping deltas is one which yields the value of the test function at _two_ points, etc.

**EDIT: The following definition is flawed. I correct it in the first comment.**

An equivalent construct in our context is an operator that takes a subset of time and maps it to an event value ("Just x") if the event has occurred within that subset. Otherwise, it maps it onto "Nothing". We can then define "sums" (or more appropriately, _mappend_) of two of these operators: map to each of the preceding values if each event has occurred in the given time subset, just like the sum of two deltas. For completeness we must also deal with combination of simultaneous events, and maybe also with the question of how to define a system that can work with both these and plain functions of time.

If we follow this plan, it's easy to see that using sums of deltas ("[impulse train](http://en.wikipedia.org/wiki/Dirac_comb)") to define event streams resembles the definition of events as functions from real time to _Maybe a_. This possibility has been suggested previously and I'm sure it's been discussed by FRP proponents - I'd like to know what conclusions or arguments were raised in this regard (**enlighten me if you know**).

However, the delta approach is not exactly the same as functions from time to _Maybe._ They are operators that take test functions and produce these _Maybe_ total functions. The type can be written as (this is not really Haskell notation):

`delta : (T -> A) -> T -> (U -> Maybe A)`

Where U is the set of subsets of T, the first argument is the test function, the second argument is the time shift of the delta - the location of the event value, and the result is a function that yields _Nothing_ everywhere except for subsets of time that happen to contain the shift value.

An event stream using this definition of delta would be a function from _subsets of time_ (not from time values) to _Maybe a_. To incorporate these events in our model we'll need can try different approaches. One way is to change the definition of a system to take inputs of this type (subsets of time → _Maybe a_), and to output a similar type. The only way I see to define the regular continuous time signals in this model is:

`f : U -> Maybe (T -> A)`

Where again U is the set of subsets of time. That is, for every _subset of_ time, the function yields a _function of_ time that gives the value of the signal at each point in the subset.

There is surely more to explore in the delta direction, but for now that's all I can think of.

# Conclusions

The idea of a measurable time domain apparently makes it possible to define both continuous and discrete timed signals and systems at once. In addition we explored the alternative route of using delta functions. The first approach (based on measurable time) for events seems simpler and more straightforward than the delta function approach. Maybe there's a simpler formulation that uses the delta approach and incorporates also regular signals in one general definition.
