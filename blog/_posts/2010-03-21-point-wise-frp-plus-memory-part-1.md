---
title: "Point-wise FRP, plus memory - Part 1"
date: "2010-03-21"
categories: 
  - "frp"
---

After [Conal's blog post](http://conal.net/blog/posts/garbage-collecting-the-semantics-of-frp/ "Conal's blog post"), I've been thinking about possible models of FRP that satisfy the "no arbitrary time access" restriction. Normally, we restrict models of time-dependent systems to causality: we disallow access to future values. The extended restriction here is about not allowing us to sample time-dependent values at any point we choose, even in the past.

We can't arbitrarily check what a sensor input was at any point in the past. We can only tell what it is _right now_, in the present. Alternatively, we can store some information about a sensor's input as time goes by, and later probe the store information to answer questions about the past. But one thing is clear: we are limited in what we can know about a past event, because our memory doesn't perfectly describe reality and it also must be finite. We can call this "point-wise time access", or maybe a better name would be "real time access".

But if we can access only the current time, how is it possible to perform things such as integration, that seem to require knowledge of the value at least at a finite time interval? Obviously the way we do it is by accumulating the sum, not by remembering what the value was at every time instant and later summing those.

As a result of that idea and a few others, I wrote the [short report titled "Behavioral Amnesia"](http://www.ee.bgu.ac.il/~noamle/_downloads/gaccum.pdf "Behavioral Amnesia"). I'd be glad to hear feedback. It discusses also other various related points.

In short, the only new (to me) idea in there is that we _can_ build a model that is point-wise (disallows arbitrary access in time) but still allows memory. One way to do so is to define a generalized operator that scans over time, I call it _scanlT_, and it works like this:

```
scanlT :: (a -> a -> DTime -> b -> b) -> [(Time,a)] -> [(Time,b)]
scanlT f y xs = scanl f' y xs'
  where f' ((t1,x1),(t2,x2)) = f x1 x2 (t2-t1)
        xs' = zip xs (tail xs)

```

Where the lists of (Time,a) and (Time,b) pairs are the time-dependent input and output values, respectively. The first argument, the function, performs the memory-full calculation. The implementation, and even the type-signature, live in the semantic domain - they describe what the model is, not how it is to be implemented.

If the time-dependent values are defined as lists, what about continuous time? The report describes a (admittedly naive) way to tackle the problem: the continuous version of _scanlT_ is the limit on _scanlT_ applied to the list of samples of the continuous input, where the limit is on the sampling rate going to infinity. I'm sure there are alternative, perhaps more general formulations - but this one suffices for me, at the present. One implication of the sampling representation is that the input must satisfy certain conditions (and if I'm correct, mainly that they be L2-integrable).

With _scanlT_ defined it's possible to work out integration:

```
integrate :: Fractional a => Temporal a -> Temporal a
integrate = scanlT psum 0
   where psum y1 y2 dt s = s + dt*(y1 + y2)/2

```

Where "Temporal a" is the yet-to-be-defined type of continuous time-dependent values. Note that there are many different formulations possible, because _psum_ is not really trapezoidal approximation - remember that _scanlT_ is taken at the limit of _dt_ going to zero. Also note that obviously a real formulation of integrals would be defined for the most general type - probably vector spaces, measurable spaces or the such (excuse my sketchy math knowledge).

As far as I know, all publicized FRP formulations to date have used integrals as built-in primitives **(correct me if I'm wrong!).** The idea I'm trying to push is that there should be nothing magic or special about integration or even general accumulation. What we want is a general way to implement realistic, non-arbitrary-time, memory.

We can also define differentiation. Differentiation is weaker in that it is local, it doesn't really need to know much about history to tell you the current value (note the \_ in the definition):

```
differentiate :: Fractional a => Temporal a -> Temporal a
differentiate = scanlT pdiff 0
   where pdiff y1 y2 dt _ = (y2-y1) / dt

```

It's fun to verify that _pdiff_ and _psum_ are inverses of each other, when you take the limit.

Finally, we can even implement maximum (or minimum). I use the imaginary typeclass "_MinBounded_" that contains the value _MinBound_:

```
maximum :: (Ord a, MinBounded a) => Temporal a -> Temporal a
maximum = scanlT pmax MinBound
   where pmax _ y2 _ m = max m y2

```

That's basically the idea, I'd love to hear comments. The [report](http://www.ee.bgu.ac.il/%7Enoamle/_downloads/gaccum.pdf "Behavioral Amensia") I wrote contains more discussion, but is currently slightly outdated only with respect to the function definitions I just gave.

In the next part I'll describe one possible model for FRP that uses this as its basis.

Lastly, comments are welcome!
