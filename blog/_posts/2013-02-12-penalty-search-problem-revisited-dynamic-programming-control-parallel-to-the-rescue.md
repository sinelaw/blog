---
title: "Penalty-search problem revisited: Dynamic Programming &amp; Control.Parallel to the rescue"
date: "2013-02-12"
categories: 
  - "haskell"
  - "math"
---

My [last post](http://noamlewis.wordpress.com/2013/02/04/finding-the-perfect-baking-time-for-that-souffle-or-optimal-search-with-test-penalty/) was about searching with a test penalty. The problem boils down to minimizing a recursive difference equation. Since I didn't (and still don't) have a closed-form solution, I had to base any conclusions on brute-force tests that find the best tree. The run time was dreadfully slow so I could calculate the solution for no more than a 15-cell array. For small arrays and a small cost constant, the solution for linear cost is just a [half-interval search tree](http://en.wikipedia.org/wiki/Binary_search).

After writing that post I realized several things and here are the new results.

As suggested by several people, the solution can be found more efficiently using dynamic programming. I've implemented a [revised version](https://github.com/sinelaw/penalty-search/blob/47728551b9231fdc075c9f0f83c218922f2f8292/test_search.hs) of the code that finds the solution by recursively finding it for subtrees. It runs much faster after this change. The DP approach also eliminates the space leak - the program uses very little memory. Also, I've added parallelization with a tiny bit of code [using parMap](http://hackage.haskell.org/packages/archive/parallel/latest/doc/html/Control-Parallel-Strategies.html#v:parMap) to cut the execution time by a factor of 4 (on my core i5). It's pretty cool how easy it is to parallelize purely functional code! After these changes, calculating an array one cell larger takes almost exactly three times longer.

Here's my theory for why three times longer. Since we split the array at the test index and have two branches for each possible root node, an array of size **n** requires solving the subtrees for arrays of sizes **1** through **n-1**, twice \[see note 1\]. So, relative to the **n-1** case we have at least twice as much work (for the two extremities that have **n-1** sized sub-arrays). Additionally we have the twice the sum from **2** to **n-1** of **2^-x** amount of work to do (relative to the **n-1** case) for the smaller sub-arrays, which converges very quickly to **1**. In total, we need to do 3 times as much work when using the DP approach.

With the improved code I've calculated the solutions for slightly larger arrays. Here are the results for the range \[0,17\] (an array of size 18):

\[caption id="attachment\_303" align="aligncenter" width="730"\][![Constant cost - binary section search](images/noname-dot.png?w=730)](images/noname-dot.png) Constant cost - binary section search\[/caption\]

\[caption id="attachment\_299" align="aligncenter" width="730"\][![Slight bias towards lower indices](images/noname-dot-2.png?w=730)](images/noname-dot-2.png) Linear cost - slight bias towards lower indices\[/caption\]

\[caption id="attachment\_300" align="aligncenter" width="730"\][![Quadratic cost - strong bias towards lower indices](images/noname-dot-3.png?w=730)](images/noname-dot-3.png) Quadratic cost - strong bias towards lower indices\[/caption\]

## Revised recommendation for confectionery research

Ok, I'll skip the soufflé analogy this time. Assuming a monotonically increasing cost function, a good strategy is what you'd expect: prioritize searching in the lower-costing area, using some sort of biased interval search (rather than half interval).

## Open question still...

I'm still looking for a closed-form solution to the cost equation. For those who didn't read the [last post](http://noamlewis.wordpress.com/2013/02/04/finding-the-perfect-baking-time-for-that-souffle-or-optimal-search-with-test-penalty/), the problem is to find the search tree that minimizes the expected cost of a search on an array, where each test carries a cost (given by a cost function). To solve the problem, one must minimize the following value by picking the optimal test point **_t_** depending on the current range _**\[x,y\]**_:

![](http://latex.codecogs.com/gif.latex?%5Clarge%20%5Cinline%20%5Cdpi%7B120%7D%20%5Cbegin%7Balign*%7D%20E%5Bcost%28x%2Cy%29%5D%20%3D%20c%28t%29%26+P%28t%20%5Cge%20s%20%7C%20s%20%5Cin%20%5Bx%2Cy%5D%29cost%28x%2Ct%29%5C%5C%20%26+%20P%28t%3Cs%7Cs%20%5Cin%20%5Bx%2Cy%5D%29cost%28t+1%2Cy%29%20%5Cend%7Balign*%7D)

The simplifying assumptions were **uniform distribution** of the search target, and **linear cost**:

![](http://latex.codecogs.com/gif.latex?\large&space;\inline&space;\dpi{120}&space;\begin{align*}&space;E[cost(x,y)]&space;&=&space;(c_0&space;t+c_1)\\&space;&+\frac{(t-x+1)cost(x,t)&space;+(y-t)cost(t+1,y)}{y-x+1}&space;\end{align*})

Another possible simplification: assume that the search algorithm uses a constant ratio to pick the next search target. That is, for a range **\[x,y\]** the next search target will always be **c \* (y - x + 1)** (rounded down) where **c** is a constant, between 0 and 1. I've toyed around with this one but ended up with the same basic problem - how to solve recursive equation?

The code is now [up on github](https://github.com/sinelaw/penalty-search) for your doubtless enjoyment.

Comments welcome!

* * *

1\. The algorithm is a bit smarter - there's no need to test the last array index, so the number of tests is slightly reduced.
