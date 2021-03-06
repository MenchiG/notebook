---
layout: post
title: Git Notebook
key: 10009
tags: tool note
category: blog
---

Some git use cases.<!--more-->

### Global settings for line endings

####  OS X or Linux

`git config --global core.autocrlf input`

#### Windows

`git config --global core.autocrlf true`

#### Refreshing a repository after changing line endings

```
git add . -u
git commit -m "Saving files before refreshing line endings"

rm .git/index
git reset
git status
git add -u
git commit -m "Normalize all the line endings"
```

### Edit Commit
	
#### Commit has not been pushed online

`git commit --amend`

#### Amending older or multiple commit messages

`git rebase -i HEAD~n`

Replace pick with `reword` before each commit message you want to change.

`git push --force`

### Squash Multiple Commits Into One

n is the number of commits that will be combined.

`git rebase -i HEAD~n`

That will open up a text-editor with something similar to the following:(from earliest to latest)

```
pick commit_1
pick commit_2
pick commit_3
...
pick commit_n
```

pick the earliest one and squash the others.

use `s` or `squash`

```
pick commit_1
s commit_2
s commit_3
...
s commit_n
```

save and quit, in a new text-editor, edit commit information.

update remote

`git push -f`  or `git push --force`



### Update a branch from master

```
git checkout master
git pull
git checkout local_branch_name
git rebase master
(may solve conflict here)
(git add conflict.FILE)
(git rebase --continue)
git push --force
```



### Overwrite branch from master

#### Be Careful !!!

abort hotfixes, and change it into master

```
git checkout hotfixes
git reset --hard master
```



### Cheat Sheets

[Atlassian](http://p2oimj6he.bkt.gdipper.com/atlassian-git-cheatsheet.pdf)
[Github](http://p2oimj6he.bkt.gdipper.com/github-git-cheat-sheet.pdf)
[Tower](http://p2oimj6he.bkt.gdipper.com/git-cheatsheet-EN-dark.pdf)