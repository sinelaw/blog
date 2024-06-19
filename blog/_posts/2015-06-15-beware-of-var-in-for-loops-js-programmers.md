---
title: "Beware of 'var' in for loops, JS programmers"
date: "2015-06-15"
categories: 
  - "javascript"
---

Time and time again, I see people using 'var' in the initialization part of a for loop. Example from [MDN (Mozilla Developer Network)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for):

```
for (var i = 0; i < 9; i++) {
   console.log(i);
   // more statements
}

```

What's wrong with `var i = 0` above? The problem is that **variables declared in a for initialization have function scope**, just like any `var` declaration does. In other words, they affect the scope of the entire function. Consider the following:

```
function outer() {
    var x = 'outer';
    function inner() {
        x = 'inner';
        //
        // ... lots o' code
        //
        for (var x = 0; x < 1; x++) {
            // in for
        }
    }
    inner();
}

```

In the inner function, `x` shadows the outer variable **throughout**, not just inside the for loop. So also the initial statement `x = 'inner'` at the head of 'inner' affects only the locally scoped variable.

This is a classic example of **[var hoisting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting)**, which should qualify as one of JavaScript's [awful parts](http://archive.oreilly.com/pub/a/javascript/excerpts/javascript-good-parts/awful-parts.html).

**Don't do it!** Move all your 'var' statements to the head of each function, please.
