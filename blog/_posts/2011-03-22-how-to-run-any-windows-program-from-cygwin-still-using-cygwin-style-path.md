---
title: "How to run any windows program from cygwin, still using cygwin-style path"
date: "2011-03-22"
---

In my previous post I posted a script useful for running kdiff3 for windows correctly from within cygwin. Here's a more general version that can be used for any windows program. Save the following as **~/cygwinify.sh** (or whatever you like):

``#!/usr/bin/bash RESULT="" for arg do if [[ "" != "$arg" ]] && [[ -e $arg ]]; then OUT=`cygpath -wa $arg` else if [[ $arg == -* ]]; then OUT=$arg else OUT="'$arg'" fi fi RESULT=$RESULT$OUT" " done echo "$RESULT"``

The script tries to find any file or directory names in your command line arguments, and converts them to absolute, windows-style path names using **cygpath -wa**. You can then run something like:

`` explorer `~/cygwinify.sh /tmp` ``

And it will open Windows Explorer in the correct folder.

Also, the script to run kdiff3 becomes (just replace the path to the executable with your own):

`` #!/usr/bin/bash "/cygdrive/d/Program Files (x86)/KDiff3/kdiff3.exe" `~/cygwinify.sh "$@"` ``
