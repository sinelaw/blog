---
title: "git admin: An alias for running git commands as a privileged SSH identity"
date: "2013-01-24"
tags: 
  - "git"
  - "ssh-key"
---

So, you want to allow some person less-privileged access to your git repository. For example, you don't want anyone pushing to master except a select few. You've got it all figured out with [gitolite](https://github.com/sitaramc/gitolite "gitolite") or whatnot by using SSH public keys as an identity that determines privileges.

Your only problem is this: ever so often you need to go over to that person's computer, sit together on some merge, and push into those higher-privileged repositories. But you can't: you're working on his/her computer, if you push you'll be using their SSH key, and you just don't have permissions.

Wouldn't it be nice if you could just type:

```
git admin push
```

...then enter your password, and get it done with? Of course it would be nice! Here's how to do it. It's not limited to "push" of course - git admin will be a prefix for any command you want to run using the alternative privileges. (And you could make many such aliases, one per SSH key you want to use - though I don't see a good use case.)

First we pick a name for the SSH key that will reside on user's computers and will be used for this purpose. Let's call it "admin".

Then:

1\. Create an SSH key **secured with a passphrase** that you (and other admins) will know, and that you will copy to the server and configure as having higher privileges:

```
$ ssh-keygen.exe -f ~/.ssh/admin
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/me/.ssh/admin.
Your public key has been saved in /c/Users/me/.ssh/admin.pub.
The key fingerprint is:
ed:1c:45:35:40:e7:e0:a6:58:a8:ce:35:e9:35:bd:00 me@PC
```

Next - we need to create a bunch of scripts to make this thing modular - let's put them all in the same place.

2\. Create a script called "ssh-as.sh" that runs stuff that uses SSH, but uses a given SSH key rather than the default:

```
#!/bin/bash
set -e
set -u

ssh -i $SSH_KEYFILE $@
```

3\. Create a script called "git-as.sh" that runs git commands using the given SSH key.

```
#!/bin/bash
set -e
set -u

SSH_KEYFILE_NAME=$1
SCRIPTS_DIR=$(dirname $0)
shift

SSH_KEYFILE=$SSH_KEYFILE_NAME GIT_SSH=$SCRIPTS_DIR/ssh-as.sh git $@
```

4\. Finally, add the alias (using something appropriate for "PATH\_TO\_SCRIPTS\_DIR" below):

```
# Run git commands as the SSH identity provided by the keyfile ~/.ssh/admin
git config --global alias.admin \!"PATH_TO_SCRIPTS_DIR/git-as.sh ~/.ssh/admin"
```

5\. Use it:

```
git checkout master
...stuff...
# Note: "git push" (to master) would fail due to insufficient privileges
#       (in our office we use gitolite to limit pushing to some branches)
git admin push
<enter password: ....>

git checkout some_branch
# Note: "git push -f" would fail due to insufficient privileges 
#       (we use gitolite to limit history rewrites in all but one's own branches)
git admin push -f
<enter password: ....>

git admin ...whatever other command that requires more privileges in gitolite or whatever...
```

etc.

We use it also for limiting destructive writes (push -f), I even limit **myself** so that if I want to push to master or do a "push -f" I must use git admin, just to remind myself what I'm doing. We use gitolite and I've created two separate identities for myself, one of which has higher privileges and uses the key used by "git admin".

That's it!

**Update:** An alternative approach using SSH .config is [given here](http://stackoverflow.com/a/7927828/562906) and basically boils down to adding multiple remotes to your git config using SSH host aliases. Each host alias uses a different SSH key, so to use different privileges one must push/pull to the different remotes. This approach is slightly simpler to set up (no scripts involved) but requires adding a new host alias to SSH for every host and also a new remote in every git repository you set up (may be nuisance if you have many repositories or, less likely, many remote hosts that you work with.)
