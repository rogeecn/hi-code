---
title: homebrew使用国内镜像源
date: 2018-04-16 00:00:00
tags: ["国内镜像"]
abbrlink: homebrew-use-china-mirror
img: ""
comments: false
---

homebrew主要分两部分：`git repo（位于GitHub`）和`二进制bottle（位于binary）`，这两者在国内访问不太顺畅。其实可以替换成国内的镜像，git repo国内镜像就比较多了，可以自行查找，如：中科大镜像…



## 替换`homebrew`默认源

### 替换brew.git:
```bash
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
```

### 替换homebrew-core.git:
```bash
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```
如果替换源之后`brew update` 没反应
```bash
cd "$(brew --repo)"
git pull origin master
```
## 如何切回官方源
### 重置brew.git:
```bash
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
```

### 重置homebrew-core.git:
```bash
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```
注释掉bash配置文件里的有关Homebrew Bottles即可恢复官方源。 重启bash或让bash重读配置文件。

## 替换Homebrew Bottles源
Homebrew Bottles是Homebrew提供的二进制代码包，目前镜像站收录了以下仓库：

- 对于bash用户
```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```
- 对于zsh用户
```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```
