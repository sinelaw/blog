---
title: "Brackets - a great editor for Javascript, HTML and CSS"
date: "2013-04-15"
categories: 
  - "javascript"
---

[http://brackets.io/](http://brackets.io/)

[![brackets](http://noamlewis.wordpress.com/wp-content/uploads/2013/04/brackets.png?w=300)](http://brackets.io)

Besides being great, it also has a fast release cycle (2.5 weeks) and is [open source.](https://github.com/adobe/brackets) It is being developed by folks at Adobe (plus others from the OSS comunity).

**Favorite features:**

- Nice hinting and inline code helpers for CSS.
- Live editing of HTML and CSS - you can fire-up a browser (currently only Chrome) that is updated as soon as you change the code, no need for F5. It also highlights the HTML your cursor is at.
- Very lightweight. Yet, it has everything you need to start working on a web project. You just install it.
- It comes with JSLint built-in that immediately bugs you about your [Javascript pitfalls](http://oreilly.com/javascript/excerpts/javascript-good-parts/awful-parts.html). Other IDEs I've used leave it to you to set up that kind of stuff.
- Extensible. The extensions I've installed include CSS color hinting, and a context-menu to open a file's folder in the OS's file system thingy (explorer, in Windows).

**Weaknesses**: a few basic editor features (such as code folding or more intelligent search) are lacking, but the basics are good enough to be very productive. Extension management is currently done via manual file copying, but according to the dev blogs, they're working on an extension manager.

**Platform support:** currently built only for Windows and Mac OSX.

**Tip:** if you're using Live Development on a site that uses AJAX to access it's own web server, you need to either enable cross-site requests on your development web server, or (easier) tell brackets where your development web server is serving your html/js/css from - it's under File...Project Settings. [See here for more.](https://github.com/adobe/brackets/wiki/How-to-Use-Brackets#live-development)
