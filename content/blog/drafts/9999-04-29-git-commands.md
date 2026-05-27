---
draft: true
title: "Everyday git commands to make your everyday life easier"
date: 2026-01-19T01:00:00+00:00
description: "There are quite a few good GUIs for git, but none of them quite matches the command line. For that reason I am always sticking to the CLI to do some more advanced stuff. "
tags: ["Git"]
---
This is a collection of all the commands I am using from time to time and started to find useful.
The collection is intended as a resource for me to lookup and extend from time to time.
I also hope to make your life easier by providing this resource.

## Reset the latest commit



``` bash
git reset --soft @~
### alternative
git reset --soft HEAD~
```

When using git reset --soft HEAD~1 you will remove the last commit from the current branch, but the file changes will stay in your working tree. Also the changes will stay on your index, so following with a git commit will create a commit with the exact same changes as the commit you "removed" before.
no add necessary afterwards

## Changing the latest commit

``` bash
git commit -m "committ with typo"
git commit -m --amend "commit without typo"
```

git commit --amend --no-edit

## Commit lines of a file only
git add -p

## Create and checkout branch
git checkout -b

## Create an empty commit
git commit -allow-empty -m "No changes made"

## Add all files and commit
git commit -am "commit message"

## Stash and rebase and push from stash
git pull --rebase --autostash

## Adjust config
git config --global user.name ""
git config --global user.email ""
git config --list

git reflog

git diff --staged