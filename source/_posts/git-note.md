---
title: Git Note
date: 2020-06-08 00:50:13
tags: git
toc: true
---

# Git Submodule

## Add Submodule

``` bash
$ git submodule add <url> <path-to-submodule>
```

Confirm submodule is added by `cat .gitmodules`.

Set git config to show submodule summary when running `git status`:

``` bash 
$ git config --global status.submoduleSummary true
```

``` bash
$ git config --global diff.submodule log
```

## Add & Commit

``` bash
$ git add .
$ git commit -m "add submodule"
```

## Clone a repo containing submodules

``` bash
$ git clone <url> <path>
```

If `cd submodules/`, nothing exists, need to do:

``` bash
$ git submodule init
$ git submodule update
```

Then `cd submodules/`, everything is okay.

## Switch submodule's version

``` bash
$ cd submodules/
$ git fetch
$ git checkout beta
$ cd ..
```

Run `git status` to see new commit in submodule.

``` bash
$ git add .
$ git commit -m "change submodule to beta version"
```

## Remove submodule

``` bash
$ git submodule deinit <path-to-submodule>
```

At this point, we can reinit this submodule.

``` bash
$ git rm <path-to-submodule>
```

Run `git status` to confirm.

``` bash
$ git add .
$ git commit -m "remove submodule"
```

## Multiple Nested Submodules

Clone a repo containing multiple nested submodules:

``` bash
$ git clone --recursive <url> <path>
```

Run `cat .gitmodules` to confirm first layer of submodules.

To look into submodules' `cat .gitmodules`, run:

``` bash
$ git submodule foreach 'cat .gitmodules'
```

## Update from submodules

``` bash
$ git pull
```

Teammates may add/delete/change/redirect a submodule. Run:

``` bash
$ git submodule sync --recursive
```

Then bring change for each submodule by:

``` bash
$ git submodule uodate --init --recursive
```

## Edit submodules

`cd submodule/`, `git pull --rebase`, make some changes, `git commit -am "edit something`, **Not to push now**.

`cd ..`, run `git status` to see new commits in submodule, then commit in **parent** repository by `git commit -am "edit submodule`

If `git push` right now, would push a commit not yet pushed in submodule. Could check each indidual submodule carefully before push in parent repo, but might miss.

In git version > 2.7, can do:

``` bash
git config --global push.recurseSubmodules on-demand
```

Then `git push`.











