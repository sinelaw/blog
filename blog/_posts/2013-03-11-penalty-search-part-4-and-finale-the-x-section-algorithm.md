---
title: "Penalty search, part 4 and finale: the x-section algorithm"
date: "2013-03-11"
categories: 
  - "haskell"
  - "math"
---

# What's the problem?

This is the last post in a series dedicated to exploring the problem of "penalty search". Penalty search involves an ordered array: the first part of the array has values that do not satisfy some property, while the second part - the rest of the values - do satisfy the property. Testing a value for the property carries an index-dependent penalty, given by a cost function. We want to find the first value (lowest index) in the array that satisfies the property. The "total cost" of a search attempt is the total penalty spent looking for the value until it is found.

The goal is to devise an algorithm that searches the array with minimal expected total cost.

# Why is this interesting?

Besides being fun to wrestle with, this problem is interesting because it deals with a realistic physical situation: you need to find out the ideal timing for some process. Any time-dependent experiment can be modeled by this problem. Every time you try, you _spend time_, so the cost of the test is time being spent. The goal is to find the solution as quickly as possible, so we want to minimize the expected time spent searching for a solution.

# Linear cost - recap

[The last post](http://noamlewis.wordpress.com/2013/02/22/penalty-search-part-3-exploring-linear-cost/) explored how a linear cost function affects the recursive difference equation that calculates the expected cost. Linear cost is interesting because first, it is easier to analyze, and second, because it fits problems where the penalty is the amount of time spent waiting to check a time-sensitive process.

Analytically, the nice property of linear cost is that the size of the array being searched is what determines the cost (up to a constant), not the specific range of indexes. In that last post we took advantage of this translational quasi-invariance of a linear cost function to simplify the problem. After defining a couple of help functions we ended with the following equation:

$latex h(N)=N(at\_N+b)+h(t\_N+1)+h(N - t\_N - 1)+a(N - t\_N - 1)(t\_N+1)$

Where:

- $latex N$ is the size of the region (array) that's being searched
- $latex h(N)$ represents the expected cost of a region (almost - you need to divide by $latex N$ to get the actual cost)
- $latex t\_N$ is the index that our algorithm decides to test in the current step
- and $latex at\_N+b$ is the cost of testing at index $latex t\_N$

# Constant ratio search

For linear cost, we expect the solution - the optimal search algorithm - to be somewhat similar to binary section search: we always pick a constant ratio ("section") of the current range to pick our next search target. Although I can't prove that this is a valid assumption, I think it's interesting enough to see what the ratio should be if we **do** assume constant ratio search.

To begin, we define $latex t\_N := \\lfloor rN \\rfloor $ where $latex r \\in \[0,1)$ (remember we are already assuming that the region is $latex \[0,N-1\]$). To make things simpler (though admittedly less precise) we'll drop the floor notation and treat $latex rN$ as if it were an integer. Substituting in the last equation, we get:

$latex h(N)=N(arN +b)+h(rN +1)+h(N - rN - 1)+a(N- rN - 1)(rN +1)$

With a little simplification (saving time using [Maxima](http://maxima.sourceforge.net/)) we get:

$latex h(N)=h(rN+1)+h((1-r)N-1)+(2ar-a r^2)N^2+(-2ar+b+a)N-a$

Notice that the two recursive references to the function $latex h$ nicely represent testing the left and right sub-regions around the split point $latex rN$. I'm personally not any closer to solving that equation, so here's where we use approximations.

In the [last post](http://noamlewis.wordpress.com/2013/02/22/penalty-search-part-3-exploring-linear-cost/) we saw that for $latex N=1$ the value must be zero. We can use this information to solve $latex N=2$:

$latex h(2)=h(2r+1)+h(2(1-r)-1)+4(2ar-ar^2)+2(a+b-2ar)-a$

Following the same logic as in that last post, we know that we can only split a region of size 2 into two subregions of size 1. This means $latex h$ must be referencing recursively calls to $latex h(1)$ twice. This limits the range of $latex r$:

$latex \\lfloor 2r \\rfloor +1=1$ and $latex 2- \\lfloor 2r \\rfloor -1=1$

Implying:

$latex \\lfloor 2r \\rfloor =0 \\implies r < 1/2$

Here we have a first result: we must split somewhere lower than the middle. _Less than_ makes sense if our linear curve is increasing, such that $latex a>0$. I didn't have time to go back and check, but I'm guessing I somewhere made that assumption implicitly along the way.

It also makes sense: if the cost is increasing linearly, the upper bound for a constant split ratio is the middle - which is exactly binary section when cost is constant (not increasing at all, $latex a=0$).

It's an interesting exercise to plug values for $latex N$ and see where the bounds on $latex r$ go. I'm leaving that for another day, or for you to try.

That's all. If you find interesting instances of this problem, let me know!
