---
title: "Git cheat sheet"
description: "Useful git command to save the day"
tags: [ "git", "cheat sheet"]
lastmod: 2017-01-17
date: "2017-01-17"
categories:
  - "Development"
slug: "git-cheat-sheet"
image: "/images/cheater.jpg"
---

### git revert
Lets say you just commited a change on a branch an you realise you are on the wrong branch. What you need to do is revert your changes stash them checkout the branch you want and re-apply your changes.

```
git reset --soft HEAD^
git stash
git checkout my_branch
git stash pop
```