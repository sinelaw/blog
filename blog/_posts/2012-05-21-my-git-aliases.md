---
title: "My Git Aliases"
date: "2012-05-21"
---

I find this stuff useful. It uses a little bash scripting (mainly to invoke commands using $()).

The 'dupes' alias is thanks to  someone's answer on StackOverflow.com - you're welcome to [click the link](http://stackoverflow.com/a/8408640/562906) upvote it.

```

# dupes - lists exact duplicates in the current branch (files that are exactly the same)
git config --global alias.dupes \!"git ls-tree -r HEAD | cut -c 13- | sort | uniq -D -w 40"

# create-branch - Creates a branch on the remote origin as well as locally, and sets up tracking
git config --global alias.create-branch '!bash -c "[[ $# = 1 ]] || echo Missing argument: new branch name && git push origin HEAD:$1 && git checkout -b $1 origin/$1"'

# commit-push - Commit and then push
git config --global alias.commit-push '!bash -c "git commit $@ && git push"'

# current-branch - prints the name of the currently checked-out branch
git config --global alias.current-branch 'rev-parse --abbrev-ref HEAD'

# latest-commit - prints the commit hash of the last commit (HEAD)
git config --global alias.latest-commit 'rev-list -n 1 HEAD'

# show-root - prints the full path to the repository's top-level
git config --global alias.show-root 'rev-parse --show-toplevel'

# delete-remote-branch - deletes a branch on the remote origin (but not locally)
git config --global alias.delete-remote-branch '!bash -c "[[ $# = 1 ]] && git push origin :$1 || echo Missing argument: remote branch name"'


```
