---
title: "Penalty search, part 3: exploring linear cost"
date: "2013-02-22"
categories: 
  - "haskell"
  - "math"
---

## Recap

The [previous post](http://noamlewis.wordpress.com/2013/02/12/penalty-search-problem-revisited-dynamic-programming-control-parallel-to-the-rescue/) - a second in a series discussing optimal search under test penalty - concluded with an open question. We can write an equation to describe the expected cost, which we want to minimize the following by picking the best $latex t\_i$ for each interval $latex \[x\_i,y\_i\]$:

$latex E\[cost(x\_i,y\_i)\] = c(t\_i) + E\[cost(x\_{i+1},y\_{i+1})\]&s=1$

The open question is: _how?_

$latex c(t\_i)$ is the test cost (penalty) function. Different values have different costs, so we expect that the optimal search will take into account the different costs and be smart about picking which values to test. As we saw previously, for constant cost the algorithm is simply binary section search. We have a recursive function of two variables (the range - $latex x\_i, y\_i$) which we want to minimize by picking yet another function (the one that picks $latex t\_i$ given the current range).

I don't know how to solve this generally, which is why I resorted to manual calculations and pretty graphs. In that same [previous post](http://noamlewis.wordpress.com/2013/02/12/penalty-search-problem-revisited-dynamic-programming-control-parallel-to-the-rescue/) I used dynamic programming - a natural problem for Haskell - to find the optimal decision tree for a specific range and cost function. Thanks to [Control.Parallel](http://hackage.haskell.org/packages/archive/parallel/latest/doc/html/Control-Parallel-Strategies.html) it was easy to improve performance by taking advantage of today's multi-core processors. What I would really like, though, is to actually find a closed-form solution, one that defines the optimal search algorithm.

## Disclaimer

I'm no mathematician (nor a computer science wizard). If you think there's a mistake anywhere, let me know.

## When you don't know, make the problem simpler

So what to do when you don't know how to solve a problem?

> Make more assumptions.TM

Hopefully, we'll learn something from the solution of the simpler problem.

### Uniform Probability

The first assumption we can make is that of uniform probability (of the search target, in the range). Here's our equation:

$latex E\[cost(x\_i,y\_i)\] = c(t\_i) + E\[cost(x\_{i+1},y\_{i+1})\]$

The range $latex \[x\_{i+1},y\_{i+1}\]$ is what our algorithm decides to search in the next step, and $latex t\_i$ is the current searched value. We expand the expectation $latex E\[\]$ of the next test (next level in the decision tree). For clarity I'll dump the indexes since we don't need them now:

$latex E\[cost(x,y)\] = c(t) + P\_{x,y}(s \\le t)E\[cost(x,t)\] + P\_{x,y}(s > t)E\[cost(t+1,y)\]$

Where $latex P\_{x,y}(A) := P(A | s \\in \[x,y\])$ is a [conditional probability](http://en.wikipedia.org/wiki/Conditional_probability). From now on I'll skip writing the expectation $latex E$. Using our assumption of uniform probability we can write:

$latex cost(x,y) = c(t) + \\frac{1}{y-x+1} \\left ((t-x+1)cost(x,t) + (y-t)cost(t+1,y) \\right )$

Notice that now our problem is _mostly_ a function of range size, so we can define $latex N\_{x,y}:=(y-x+1)$ and write:

$latex cost(x,y) = c(t) + \\frac{1}{N\_{x,y}}(N\_{x,t}cost(x,t) + N\_{t+1,y}cost(t+1,y))$

Multiply by $latex N\_{x,y}$ to get:

$latex N\_{x,y}cost(x,y) = N\_{x,y}c(t) + N\_{x,t}cost(x,t) + N\_{t+1,y}cost(t+1,y)$

Being a non-mathematician, here's where I said "Aha!". We can make things simpler if we define:

$latex g(x,y) := N\_{x,y}cost(x,y)$

Since $g$ is just $cost$ multiplied by $y-x+1$, minimizing $g$ will also minimize $cost$. Using this definition the equation becomes:

$latex g(x,y) = N\_{x,y}c(t) + g(x,t) + g(t+1,y)$

...which is beginning to look a lot more manageable. And remember - so far, the only assumption we've made is that of uniform probability. But how to proceed? We still have a recursive equation with two variables, x and y.

### From two variables to one

Someone must have once said: "one is better than two." Variables. In a recursive function.

Let's add another assumption - that the cost function is linear:

$latex c(t) = at+b$

Where $latex a$ and $latex b$ are arbitrary constants. Why linear? This cost function has the nice property that the cost of any search tree is **invariant under translation** (of the searched range) up to a constant addition. That is,

$latex cost(x,y)+a u=cost(x+u,y+u)$

or

$latex g(x,y)+N\_{x,y} a u = g(x+u,y+u)$

How do I know this? Here's an informal proof. A search tree consists of nodes representing the values being tested (and cost being spent). If we translate the underlying range, we're in effect changing the linear cost function by adding a constant that depends on the amount of translation. So the cost of each node in the search tree will vary by the same constant, leaving the distribution of cost throughout the tree equivalent and keeping the expected cost minimal compared to other trees on the same range.

This property means that given a range size $latex N$ we can find the best search tree for just one translation of that range (say, $latex x=0$) and it will be correct for _any_ other range of the same size.

Plugging in the linear cost in the equation from before gives us:

$latex g(x\_i,y\_i) = N\_i a t\_i + N\_i b + g(x\_i,t\_i) + g(t\_i+1,y\_i)$

I'm explicitly writing that the choice of $latex t$ depends on $latex x,y$, just to make things clearer. We'll use the indexed notation $latex t\_i$ to signify that $latex t$ depends on the $latex i$th range, and use it also for the range size, that is $latex N\_i := N\_{x\_i,y\_i}$.

Now we can use the translation invariance to replace $latex x\_i$ everywhere with 0. At the same time we can get rid of $latex y\_i$ since it will be equivalent to make the function depend on $latex N\_i=y\_i+1$. To that end, define:

$latex h(N) := g(0,N-1) = g(x,x+N-1)-Nax = N(cost(x,x+N-1)-ax)$

Then, the equation above for $latex g(x\_i,y\_i)$ becomes:

$latex h(N)=Nat\_N+Nb+h(t\_N+1)+h(N - t\_N - 1)+a(N - t\_N - 1)(t\_N+1)$

That last term is due to the required translation of the $latex h(N-t-1)$ from the origin to $latex t+1$ (the original value was $latex g(t\_i+1,y\_i)$.

## Results please...

Let's look at the actual value of the expected cost for a few small N's (region sizes):

### N=1

When the size of the region, $latex N$ is 1, there is nothing to check - by assumption the target value is in the region and there is only one cell, which must be it. Since there is nothing to check, there is no cost, so $cost(0,1)=0=h(1)$. Also, we don't care what the offset (translation) of this one-celled region is - so we don't need to add the additive term when calculating translated calls such as cost(x,x+1) because the value is always 0.

### N=2

Let's move on to $latex N=2$. For a region of two cells, the only logical algorithm is to check the first cell. If it yields the value 0, then the target (first post-step cell) is cell #2, otherwise (value at first cell is 1) - the target is cell #1. So the expected cost is just the cost of the first cell - which has index 0 - which is $0a+b=b=cost(0,1)$.

That's what simple logic dictates. Now let's see how our equation fairs:

$latex h(2) = 2at+2b+h(t+1)+(h(2-t-1)+a(2-t-1)(t+1))$

Remember that $latex t \\in \[0,N-1\]$, so in this case t can only have the value 0 (matches our logic that we can only test cell 0). Since the second recursion ($latex h(2-t-1)$) turns out to be a 1-celled region, we delete the additive term and replace it with 0:

$latex h(2) = 2b+h(1)+h(1)=2b$

Since $latex h(2)=2cost(0,1)=2b$ the result matches our "manual logic" from above.

### N=3

Here's where things start to get interesting. $latex N=3$ is the first case where we have to compare different values for $latex t$ to find out which one yields the minimal expected cost. The possible values are 0 and 1:

- $latex t=0 \\Rightarrow h(3)=3b+h(1)+h(2)+2a=3b+0+2b+2a=2a+5b$
- $latex t=1 \\Rightarrow h(3)=3a+3b+h(2)+h(1)+0=3a+3b$

So which is less? We should pick $latex t=0$ iff $latex 2b \\le a$. Otherwise, pick $latex t=1$.

This is interesting because it means that the optimal search algorithm depends on how our linear cost is defined. If the slope $latex a$ is "twice more dominant" than the constant $latex b$, then we should bias our search towards 0. Otherwise, we go with binary section and take the middle cell with index 1.

# What's next

We could spend the rest of our lives calculating larger and larger values of N. What we really want, though, is a general, closed-form solution for any N. In my next post I'll make another weakening assumption that will make it possible.
