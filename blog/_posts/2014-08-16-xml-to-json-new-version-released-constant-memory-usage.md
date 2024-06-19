---
title: "xml-to-json - new version released, constant memory usage"
date: "2014-08-16"
categories: 
  - "haskell"
---

I've released a new version (1.0.0) of [xml-to-json](https://github.com/sinelaw/xml-to-json), which aims to solve memory issues encountered when converting large XML files. The new version includes two executables: the regular (aka "classic") version, xml-to-json, which includes the various features, and the newly added executable xml-to-json-fast, which runs with constant memory usage and can process files of arbitrary size. It does this by not validating the input xml, and by basically streaming json output as it encounters xml elements (tags) in the input. The implementation takes advantage of the cool [tagsoup](https://hackage.haskell.org/package/tagsoup) library for processing XML.

Check the [README.md](https://github.com/sinelaw/xml-to-json/blob/master/README.md) for more details. Hackage is updated.
