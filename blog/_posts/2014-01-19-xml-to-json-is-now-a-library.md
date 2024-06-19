---
title: "xml-to-json is now a library"
date: "2014-01-19"
categories: 
  - "haskell"
  - "javascript"
---

A while back I needed to convert a ton (millions) of small xml files to json, so I could store them in [MongoDB](http://www.mongodb.org/). To that end I wrote a teensy-tiny tool called xml-to-json ([github](https://github.com/sinelaw/xml-to-json), [Hackage](http://hackage.haskell.org/package/xml-to-json)). Originally it was just a command-line tool with all the code thrown in a single file.

So, I did a quick refactor this week to split it into a library + executable, and pushed it to github (to deafening cries of joy).

## Features

First, a non-feature. xml-to-json is "optimized" for many small xml files. If you have many small xml files, you can easily take advantage of multiple cores / cpu's. You should be aware that for large files (over 10MB of xml data in a single file) something starts to eat up RAM, around 50 times the size of the file.

Other features:

- You can filter xml subtrees to convert, by element name regex (and you can skip the matching tree root if you wish, converting only the child elements and down).
- Output either a top-level json object or json array.
- (Optionally) simplify representation of xml text nodes in attribute-less elements (e.g. "<elem>test</elem>" -> { elem: "test" })

## Packages used

For XML decoding, I'm using [hxt](http://hackage.haskell.org/package/hxt) (over [expat](http://expat.sourceforge.net/) using [hxt-expat](https://hackage.haskell.org/package/hxt-expat)). I tried a few of the xml packages on hackage, and hxt + expat was the only way I could parse quickly while avoiding [nasty memory issues](http://stackoverflow.com/questions/2292729/with-haskell-how-do-i-process-large-volumes-of-xml). Apparently, tagsoup can be used with Bytestrings to avoid the same issue but I didn't try.[](http://stackoverflow.com/questions/2292729/with-haskell-how-do-i-process-large-volumes-of-xml)

JSON is encoded using [aeson.](http://hackage.haskell.org/package/aeson)
