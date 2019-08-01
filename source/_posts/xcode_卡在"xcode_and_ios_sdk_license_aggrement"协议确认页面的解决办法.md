---
title: xcode 卡在"xcode and ios sdk license aggrement"协议确认页面的解决办法
date: 2018-04-18 00:00:00
tags: ["macOS"]
abbrlink: resolve-xcode-block-at-xcode-and-ios-sdk-license-aggrement-dialog
img: ""
comments: false
---

当前老系统，新装完 "macOS Mojava", Xcode 下载完后直接打开应用弹出协议确认框，点`Agree`按钮死活没反应，使用下面的命令行办法解决。
1. 打开命令行
2. 运行 `xcodebuild -h`, 如果有提示 `xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance`，运行这个命令切换XCODE目录`sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/`，需要输入密码。
4. 运行`sudo xcodebuild -license`点回车后会出来一大段协议内容，一直按空格直接出现输入框提示确认，输入`agree`点确认即可。
5. 现在可以直接打开xcode了。
