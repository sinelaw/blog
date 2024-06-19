---
title: "Finding the perfect baking time for that soufflé (or: optimal search with test penalty)"
date: "2013-02-04"
categories: 
  - "haskell"
  - "math"
---

## TL;DR

How to find a value in a range of integers where every test carries a cost with pretty graphs at the end.

## Don't ruin that soufflé

You know that problem. If you take it out of the oven too soon, and you get a raw egg; leave it in for too long, and you'll end up with a fat, flat omelet (or is it omelette?). A soufflé is like Schrödinger's cat: if you open the oven to observe, it will collapse from the good-bad superposition into a slobbery glob of goo. So what to do?

The real answer is to get a good cookbook or tips from someone who knows. But let's not bother with reality here: there's an interesting problem hiding here that's worth exploring. Let's assume you can only make one soufflé at a time, and that if you open the oven to check, you can't put it back in. It will either be underdone, exactly ready, or overdone. Also let's assume that the ideal time to bake a soufflé is a constant, integer number of minutes. You need to find out quickly what's the best baking time because you are hosting a party in a few hours and want to be prepared. Finally, let's assume that we tried to make one and after baking it for 30 minutes found that it was overdone.

So what's the _fastest_ way to find out the perfect baking time for a soufflé?

## So here's the actual question

To state the problem more generally: we have a test, a boolean function of integers _**f(t)**_, that tells us whether a desired value _**s**_ is less than the tested value _**t**_. Or, in Haskell:

```
f :: Integer -> Bool
f t = t >= s
```

Every time we do the test (evaluate _**f(t)**_) we pay a penalty - in the case of a soufflé, we waste time: we have to restart the process and start baking a new one. Let's call this penalty the _cost function_, and let it depend on the value we are testing. For our simple purposes, let the function be from integers to integers.

```
cost :: Integer -> Integer
```

Finally, we point out that _**s**_, that value we're trying to find, is a random variable with probability function _**pf**_ for finding _**s**_ in a range _**\[x,y\]**_. This function satisfies:

```
pf :: Integer -> Integer -> Integer -> Double
-- the rule is:
pf x y z = sum . map pmf $ [x..y]
```

Where _**pmf**_ (probability mass function) is a distribution on the total range _**\[a,b\]**_ where _**s**_ is known to reside.

```
pmf :: Integer -> Double
```

Being a probability mass function, the rule that _**pmf**_ must hold is:

```
sum . map pmf $ [a..b] = 1
-- or
pf a b = 1
```

The problem is then, to find an algorithm that **finds _s_ with minimal expected cost**. The average total cost of checking a range _**\[x..y\]**_ when testing a value _**t**_ in the range, can be calculated recursively. Let's say our algorithm decides which value to test next using a function _**guessVal(x,y)**_. First we need to define the conditional probability _**cpf**_ of finding _**s**_ in a range _**\[x',y'\]**_ given the fact that it is know to lie in an extended range _**\[x,y\]**_:

```
cpf :: Integer -> Integer -> Integer -> Integer -> Double
cpf x y x' y' = ... -- related to pf and depends on the distribution
```

Then we can write:

```
avgCost :: Integer -> Integer -> Double
avgCost x y = 
    if (x == y) 
    then 0 
    else cost t + (cpf' x t) * (avgCost x t) 
                + (cpf' (t+1) y) * (avgCost (t+1) y)
    where t    = guessVal x y
          cpf' = cpf x y
```

This assumes that _**t**_ is actually in the range _**\[x,y\]**_ and that _**s**_ is known to be in that range. If the range is a single integer, there is nothing to check and thus zero cost. Otherwise, the cost is the probability-weighted average of the two next branches (the guessed value yielded a test result false or true). Note that for any distribution, the total of the two _**cpf**_ values above will always be 1 since we are covering the entire range on which the condition is applied. We need to minimize the expected cost, so the general problem is:

> Choose _**guessVal**_ that minimizes _**avgCost**_ for a given range _**\[a,b\].**_

A minor subtlety to note is that we never need to test the highest value of a range because it will always test positive. This can be expressed as the rule:

```
x <= (guessVal x y) < y
```

### Simplifying assumption

Let's assume that our probability distribution is uniform. In that case the probability function depends only on the size of the range being tested and a constant that depends on the complete range size:

```
pf x y = (y - x + 1) / (b - a + 1)
-- and conditional probability:
cpf x y x' y' = (y' - x' + 1) / (y - x + 1)
```

Under these conditions the avgCost function may be rewritten as:

```
avgCost x y = 
    if (x == y) 
    then 0 
    else cost t + c * ((t - x + 1) * (avgCost x t) 
                         + (y - t) * (avgCost (t+1) y))
    where t = guessVal x y
          c = 1.0 / (y - x + 1)
```

In many cases uniform distribution is not a good pick - for example, when we have a pretty good guess about where _**s**_ lies in the range a normal distribution would be more appropriate.

## Cost functions

#### Constant cost

In the trivial case, the cost function is just a constant - most trivially, it is always 1. In that case our problem boils down to minimizing the **number of tests** and is the same as finding a value in an array. The well-known solution to that is **binary search**. Our average cost is then:

```
avgCost x y = 
    if (x == y) 
    then 0 
    else 1 + 2 * 1/2 * (avgCost x t)
    where t    = x + (y - x) / 2
```

This reduces to the expected value of _**log(n)**_ (rounded up) where _**n**_ is the size of the complete range.

#### Linear cost

Unfortunately, when baking a soufflé every attempt has more than a constant cost - you have to wait a certain amount of time to find out if a given duration for baking is too short or too long. If time is of the essence, an appropriate cost function would be **linear and increasing**, such as _**cost(t) = t**_. In that case the equation becomes:

```
avgCost x y =
    if (x == y)
    then 0
    else t + c * ((t - x + 1) * (avgCost x t) + (y - t) * (avgCost (t+1) y))
    where t = guessVal x y
    c = 1.0 / (y - x + 1)
```

Just a reminder: the problem is to minimize the average cost by picking the right **guessVal** function - picking the right t every time. If anyone solves this, let me know.

## Visualization

As suggested by [this answer](http://cstheory.stackexchange.com/a/16223/13449)1, any sensible algorithm is equivalent to a binary decision tree (given a range _**\[a,b\]**_). At each node, the algorithm tests a value _**x**_, spending _**cost(x)**_, to determine which of the branches to take. If we assign a cost value to each node in the tree, minimal expected cost is the weighted average root-to-leaf total cost (where the weights depend on the distribution _**pmf**_; for uniform distribution the weights are identical and can be ignored.) The leaves themselves have zero cost because they signify that the result is known.

I wrote [a small program](https://gist.github.com/4704827) that finds the best algorithm (decision tree) for a given range and cost function, and prints out the results in graphviz-compatible .dot format. Here they are for constant cost (= binary search), linear and quadratic costs for the range \[0,14\]. Since I don't (yet) have a solution for the linear cost case, I was surprised to find that the best algorithm for **linear cost is still binary search** (**update:** **that's not true, this was a side effect of testing only small trees - I'll write a new post about the updated analysis**) despite having what is intuitively a considerable bias toward the beginning of the range. For quadratic cost (_**cost(t) ~= t^2**_), the best algorithm is quite lopsided as you can see below. It's possible that there's a slight bias effect also for linear cost that is only prevalent for larger intervals.

\[caption id="attachment\_274" align="aligncenter" width="630"\][![Linear cost yields the same graph as constant cost](images/constant-14.png?w=630)](images/constant-14.png) Linear cost yields the same graph as constant cost for the range \[0,14\]\[/caption\]

\[caption id="attachment\_275" align="aligncenter" width="630"\][![Quadratic cost yields a lopsided graph](images/quadratic-14.png?w=630)](images/quadratic-14.png) Quadratic cost yields a lopsided graph\[/caption\]

## The dreaded Space Leak

The range is that small because the number of trees to test [increases very fast](http://oeis.org/A000108) - I could only get up to 14 before running out of memory, which is another story. The implementation is naive: the program builds all the trees before calculating the average cost of each - the right way would be to calculate the cost of each tree and discard it so that the GC could get rid of it when memory gets tight.

## So what about that soufflé?

As the visualized in the graph for linear cost above, when time is the issue **you should** **still use binary section search. (update: next post will explain why this is wrong).** If the cost is quadratic, give higher priority to lower values.

## Open questions

1. Solve the minimization problem for the recursive equation for _**avgCost**_ when the cost function is linear: _**cost(t) = t**_.
2. Take into consideration other probability distributions. For example, solve for linearly increasing distribution (where larger values are more likely to equal _**s**_) and normal distribution around the middle of the range.
3. How do I improve the implementation of the brute force algorithm to not explode in RAM?

Let me know when you're done :)

If you have any comments or insights, I'd be glad to hear them! My family can take a few more soufflé attempts...

* * *

1\. Since I couldn't find any information about this problem I [asked this question](http://cstheory.stackexchange.com/questions/16216/searching-for-the-first-item-satisfying-property-with-penalty-for-every-test) on [cstheory.stackexchange.com](http://cstheory.stackexchange.com). I got some backlash from one person about this not being a "research-level question" as dictated by the site's FAQ... You can check out the other questions on the site to decide for yourself if this question exceeds the maximum dumbness level.
