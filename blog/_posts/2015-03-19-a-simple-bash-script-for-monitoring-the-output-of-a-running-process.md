---
title: "A simple bash script for monitoring the output of a running process"
date: "2015-03-19"
---

[![tailor.sh example](images/ezgif-com-crop.gif)](https://noamlewis.wordpress.com/wp-content/uploads/2015/03/ezgif-com-crop.gif)I got tired of my screen getting flooded with text when all I wanted was to see the last output line **at any given time**. What I really want is to just see one output line and have it replaced with some newer text when it's available. So I wrote a script (in, ahem, bash) that lets your do exactly this. Call it "tailor.sh" or whatever you want:

(If the code below is screwed up, [see it here on gist](https://gist.github.com/sinelaw/12b7957950c5103e7b46))

```
#!/bin/bash -eu
LOG_FILE=$1
SB="stdbuf -i0 -oL"
shift
tput sc
$@ 2>&1 \
   | $SB tee $LOG_FILE \
   | $SB cut -c-$(tput cols) \
   | $SB sed -u 's/\(.\)/\\\1/g' \
   | $SB xargs -0 -d'\n' -iyosi  -n1  bash -c 'tput rc;tput el; printf "\r%s" yosi'
EXIT_CODE=${PIPESTATUS[0]}
tput rc;tput el;printf "\r" # Delete the last printed line
exit $EXIT_CODE

```

I use it in a test env bringup script to run some builds. Instead of having to lose all the context of the wrapping script, I now see just a single output line of the invoked builds at any given time.
