---
title: "cabal install FTGL on Windows, and getting exe's that use it to work"
date: "2012-12-16"
categories: 
  - "haskell"
---

I wanted to try out the latest version of [lamdu, a "live programming" environment](https://github.com/Peaker/lamdu) (still in early development). It uses [graphics-drawingcombinators](http://hackage.haskell.org/package/graphics-drawingcombinators) which in turn depends on [FTGL](http://hackage.haskell.org/package/FTGL) to accomplish its text-drawing-in-OpenGL magic. On linux this isn't really an issue - a simple 'cabal install' does it, at least on the version of ubuntu that I use (EDIT: You'll probably first need to install the ftgl dev files, e.g. `sudo apt-get -y install libftgl-dev`)

Windows? No problem! With a few hacks, you'll be rendering text in no time. This worked on my 64-bit Windows 8 installation but should work on any version since Windows XP.

1. I'm assuming you have the latest [Haskell Platform for Windows](http://www.haskell.org/platform/windows.html) installed. If not, do it!
2. Get 32-bit windows binaries for FreeType and FTGL. I downloaded them from: [http://www.opencascade.org/getocc/download/3rdparty/](http://www.opencascade.org/getocc/download/3rdparty/), but you might as well compile them from the official sources.
3. Copy the FTGL.dll and FreeType.dll to:
    1. 64-bit version of Windows: copy to c:\\windows\\syswow64
    2. 32-bit version of Windows: copy to c:\\windows\\system32
4. Install the [Visual C++ 2010 redistributable, 32-bit version](http://www.microsoft.com/en-us/download/details.aspx?id=5555)
5. Assuming you've unpackged the FTGL binaries in some directory "<blabla>\\ftgl-2.1.3-vc10-32", run the following:`cabal install ftgl --extra-include-dirs=<blabla>\ftgl-2.1.3-vc10-32\include --extra-lib-dirs=<blabla>\ftgl-2.1.3-vc10-32\lib --reinstall --force-reinstalls`
6. cabal build / install whatever executable you wanted to (in my case, lamdu)

That's it! Hope I saved someone the near-hour I spent figuring this out.
