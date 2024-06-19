---
title: "Introducing PopSimple: An experiment in web content mashup"
date: "2012-09-19"
---

[![Image](http://noamlewis.wordpress.com/wp-content/uploads/2012/09/small-logo.png?w=26)PopSimple.com](http://www.popsimple.com/)

[![PopSimple - Screenshot](images/popsimple_screenshot.png "PopSimple - Screenshot")](http://www.popsimple.com)

Hadas Nahon and myself have been working on a project for a couple of months last year, with the intention of building a generic UI for "throwing stuff on a page and sharing it." It runs without plugins in any modern browser. Due to previous commitments (and family-related considerations) we've never had the chance to really finish the project, so I decided to release it as it is. It's running at [PopSimple.com](http://www.popsimple.com/) and the [full source](https://github.com/popsimple/popsimple) is at github for your forking leisure.

## Current features

- Content tools - allow you to place content on the page:
    - Rich text
    - Images
    - Videos
    - Maps
    - and even drawings (as far as your mouse-based artistic talents can take you)
- Free-form: rotate, scale and drag content on the worksheet to visually arrange your content
- Share an editable link or a read-only "view" link to the page you created

A few technical details about the nature of the project follow.

## How to write a big fat rich web client?

As mentioned, it's a pure browser-based implementation, which happens to mean running in JavaScript. However it was NOT written in JavaScript - rather, almost all the code is in Java using [Google Web Toolkit (GWT)](https://developers.google.com/web-toolkit/). Besides the (enormous) advantage of writing statically-typed code in Java, other nice features of GWT are that it takes care of packing your resources - images and CSS - in an optimally minimal way, and that it integrates easily with Java-based server side code. With all it's awesomeness, GWT seems to be losing Google's focus (just look at the decreasing frequency of official blog posts, not to mention releases). In my company we were recently debating whether to use GWT as a platform for some web based projects, and decided not to, just to avoid a case of "abandoned by Google." Still, in my opinion it's a cool piece of technology which is in desperate need of standardization.

Did I mention it runs on [Google App Engine (GAE)](https://developers.google.com/appengine/)? There are only two things we need the app engine for: saving and loading pages. Everything else the application does is completely browser based. We have some conclusions about the pros and cons of GAE, but I'll leave that for another time. The main reason we use GAE is because it made deploying our site much easier (and it was free).

For a nicer-than-metal interface to GAE we use [Twig](http://code.google.com/p/twig-persist/), a nifty library that provides an object-level persistence API to the app engine data store.

## Modern browsers, please

We decided to assume that the browser you're using is fairly modern - taking advantage of CSS3 and even a teeny bit of HTML5 (for the drawing tool - uses a canvas, and for dropping image files onto the worksheet).

## Other stuff

Some of the cooler implementation details include a framework for chaining asynchronous functions (a la ["futures" or "promises"](http://en.wikipedia.org/wiki/Promise_%28programming%29) or [the new async/await C# features](http://msdn.microsoft.com/en-us/library/hh191443.aspx)). We use those for running stuff after dynamically loading some javascript libraries (such as google maps) while avoiding "callback hell". Also we've made use of [Mapstraction](http://mapstraction.com/) - a library that abstracts away web-based maps APIs and wraps them nicely (we make use of Google, Microsoft and OpenLayers via a single API).

I hope to blog again about how we built PopSimple, I'm trying to decide which of the problems we tackled could be interesting to write about.

## Check it out!

The project is up & running at [PopSimple.com](http://www.popsimple.com/) - so check it out! We'd love to know what you think. You can even open issues on the [github issues page](https://github.com/popsimple/popsimple/issues/new) for the project if you feel like it.
