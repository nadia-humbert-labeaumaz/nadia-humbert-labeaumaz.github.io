---
type: "post"
title: "Go Back in Time with Git"
date: 2017-04-19 20:22:15 +0100
description: ""
author: Nadia Humbert-Labeaumaz
image : "images/blog/back-in-time-git/header.jpg"
bg_image: "images/blog/back-in-time-git/header.jpg"
categories: ["Software"]
draft: false
comments: true
---

Recently, I conducted a workshop about how to go back in time with Git alongside [Renaud](https://twitter.com/rnowif). Here are the main points that were raised during this session.

<!-- more -->

## Case #1: Delete the Last Commit

The initial Git tree used to illustrate this case is:

```bash
* 7ec8248 N - (HEAD -> master) Hello, world!
* 26af837 N - Hello, world
* c9b1299 N - Hello
```

The goal here is to delete the last commit so the resulting tree looks like this:

```bash
* 26af837 N - (HEAD -> master) Hello, world
* c9b1299 N - Hello
```

The easiest way to achieve this purpose is to use the `git reset` command.

```bash
$ git reset --hard 26af837
```

It sets the current branch to the specified commit. The `--hard` option will discard all changes that have been made after the specified commit.

## Case #2: Create a Branch from a Previous Commit

In this case, the initial Git tree is the same as before.

```bash
* 7ec8248 N - (HEAD -> master) Hello, world!
* 26af837 N - Hello, world
* c9b1299 N - Hello
```

We want to create a branch from the commit `26af837` to fix a bug for instance. The resulting tree should be the following:

```bash
* ae77cf0 N - (HEAD -> bug-fix) Fixed it!
| * 7ec8248 N - (master) Hello, world!
|/  
* 26af837 N - Hello, world
* c9b1299 N - Hello
```

First, we need to position ourselves on the commit:

```bash
$ git checkout 26af837
```

Then, we are in the 'detached HEAD' state which means that we are no longer on a branch and further commits will not be kept. Consequently, we create the branch and commit the bug fix:

```bash
$ git checkout -b bug-fix
$ # Fix the bug ...
$ git commit -m "Fixed it!"
```

## Case #3: Put the Last Commit on a New Branch

Here, we made a commit on the `master` branch by mistake and we want to transfer it to another branch.

The Git tree should go from this:

```bash
* 7ec8248 N - (HEAD -> master) Hello, world!
* 26af837 N - Hello, world
* c9b1299 N - Hello
```

to this:

```bash
* 7ec8248 N - (HEAD -> feature) Hello, world!
* 26af837 N - (master) Hello, world
* c9b1299 N - Hello
```

To do so, we just have to create the `feature` branch from the last commit and reset `master` to the previous one.

```bash
$ git checkout -b feature
$ git checkout master
$ git reset --hard 26af837
$ git checkout feature
```

## Case #4: Rewrite History

In this case, we want to completely remove a past commit from the Git tree.

The initial tree looks like this:

```bash
* b1b5f0a N - (HEAD -> master) Hello, world
* b7d38ac N - Add key.txt
* c9b1299 N - Hello
```

We want the resulting tree to show no sign of the commit `b7d38ac`:

```bash
* efb44c9 N - (HEAD -> master) Hello, world
* c9b1299 N - Hello
```

The `git rebase -i` command allows to rewrite the history of a branch from a specific starting point. It will prompt the list of all the commits since this starting point. You can perform different actions on these commits like reword a commit message, squash several commits into one, reordering and delete commits for instance. After these modifications, lines will be executed from top to bottom. More information about rewriting history can be found [here](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History).

```bash
$ git rebase -i HEAD~2
$ # Delete the line corresponding to the commit to remove.
```

Since we wanted to go two commits back, we used the notation `HEAD~2` to specify the starting point. Note that we could also have used the specific hash of the commit (`c9b1299`).

## Case #5: Revert a Commit

During this workshop, an attendee talked about the `git revert` command. Unlike the other commands that we saw in this article, the `git revert` command do not modify past commits. It creates a **new** commit that is the exact opposite of the reverted commit.

For instance, if we start from this Git tree:

```bash
* f999291 N - (HEAD -> master) Hello, world
* a9f8fb3 N - Add key.txt
* c9b1299 N - Hello
```

and revert the commit a9f8fb3 which contains only a new file, a new commit that only contains the removal of this file will be created.

```bash
$ git revert a9f8fb3
```

```bash
* 55bc161 N - (HEAD -> master) Revert "Add key.txt"
* f999291 N - Hello, world
* a9f8fb3 N - Add key.txt
* c9b1299 N - Hello
```

I think that this command could be used when several people are working on the branch in order to avoid a forced push on the remote repository. Moreover, it could also be used if you want to keep an explicit trace of this action in the tree for a particular reason.

## Last resort: Keep Calm and Use Git Reflog

Renaud wrote an [article](https://blog.crafties.fr/2017/04/11/git-reflog/) on the `git reflog` command to recover commits that appear to be lost. To illustrate what this command does, here is its ouput when used in the case #4:

```bash
$ git reflog
efb44c9 HEAD@{0}: rebase -i (finish): returning to refs/heads/master
efb44c9 HEAD@{1}: rebase -i (pick): Hello, world
c9b1299 HEAD@{2}: rebase -i (start): checkout HEAD~2
26af837 HEAD@{3}: commit: Hello, world
712d068 HEAD@{4}: commit: Add key.txt
c9b1299 HEAD@{5}: commit (initial): Hello
```

You can see that the deleted commit (`712d068`) still appears in the reflog.

## Conclusion

This article shows several ways to go back in time with Git. These commands could be used alone or combined to get you out of complicated situations or to rearrange your Git tree before a `git push` for instance.
