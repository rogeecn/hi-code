---
title: Git忽略已经被提交的文件
date: 2018-01-24 00:00:00
tags: ["Git"]
abbrlink: git-ignore-commited-file
img: ""
comments: false
---

.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。
正确的做法是在每个clone下来的仓库中手动设置不要检查特定文件的更改情况。

```
git update-index --assume-unchanged PATH
```
在PATH处输入要忽略的文件。



另外 git 还提供了另一种 exclude 的方式来做同样的事情，不同的是 .gitignore 这个文件本身会提交到版本库中去。用来保存的是公共的需要排除的文件。而 .git/info/exclude 这里设置的则是你自己本地需要排除的文件。 他不会影响到其他人。也不会提交到版本库中去。

.gitignore 还有个有意思的小功能， 一个空的 .gitignore 文件 可以当作是一个 placeholder 。当你需要为项目创建一个空的 log 目录时， 这就变的很有用。 你可以创建一个 log 目录 在里面放置一个空的 .gitignore 文件。这样当你 clone 这个 repo 的时候 git 会自动的创建好一个空的 log 目录了。
