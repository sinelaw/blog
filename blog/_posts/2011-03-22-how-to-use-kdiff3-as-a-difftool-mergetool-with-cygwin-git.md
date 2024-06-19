---
title: "How to use kdiff3 as a difftool / mergetool with cygwin git"
date: "2011-03-22"
---

First, install kdiff3 for windows.

Second, create the following script somewhere (such as ~/kdiff3.sh, and change the location of your kdiff3.exe to an appropriate cygwin-style path to where you executable really is). The script is based on [Pete Goodliffe's one he used for svn](http://goodliffe.blogspot.com/2009/04/subversion-kdiff3-and-cygwin.html?showComment=1300806241031). I've expanded on the idea to make it more flexible. **See my [next post](http://noamlewis.wordpress.com/2011/03/22/how-to-run-any-windows-program-from-cygwin-still-using-cygwin-style-path/) for a more general version that can be used for any windows program**

``#!/bin/sh RESULT="" for arg do if [[ "" != "$arg" ]] && [[ -e $arg ]]; then OUT=`cygpath -wa $arg` else OUT=$arg if [[ $arg == -* ]]; then OUT=$arg else OUT="'$arg'" fi fi RESULT=$RESULT" "$OUT done /cygdrive/d/Program\ Files\ \(x86\)/KDiff3/kdiff3.exe $RESULT``

Finally, configure your ~/.gitconfig to: `[diff] tool = kdiff3 [merge] tool = kdiff3 [mergetool "kdiff3"] path = ~/kdiff3.sh keepBackup = false trustExitCode = false`

No thanks to wordpress for ruining the whitespace formatting... :(

Try running **git difftool** (make sure you have some modified files) to make sure it works.
