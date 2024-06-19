---
title: "The wl-pprint package maintainer..."
date: "2015-04-14"
---

...is now me.

I've talked to the previous maintainer who is no longer interested. There was a small change required to support the new ghc (7.10), so I also released a new version. Since the new version only benefits people who need a new ghc, I also added the **source-repository** field in the cabal file, despite it breaking cabals older than 1.6.

According to hackage, [wl-pprint](https://hackage.haskell.org/package/wl-pprint) was authored by [Daan Leijen](http://research.microsoft.com/en-us/people/daan/). The new github repo is up at [https://github.com/sinelaw/wl-pprint](https://github.com/sinelaw/wl-pprint). I've taken the liberty of forking based on someone else's patch for GHC 7.10 - thank you [Alex Legg](https://github.com/alexlegg)!

Incidentally, on hackage, there are several packages with the substring 'wl-pprint' in their names, some are newer and better maintained and have more features than others. This situation is rather confusing. wl-pprint itself is being used as a dependency by at least one package I need (language-ecmascript).

It would be nice if the community converged on a single Wadler-pretty-printer package. (Probably not wl-pprint - maybe [ansi-wl-pprint](https://hackage.haskell.org/package/ansi-wl-pprint)?)
