---
title: MAC禁用Adobe Creative Cloud自启状态栏
date: 2018-04-17 00:00:00
tags: ["MAC"]
abbrlink: mac-disable-adobe-creative-cloud-auto-startup-on-status-bar
img: ""
comments: false
---

禁用Creative Cloud自启

```bash
launchctl unload -w /Library/LaunchAgents/com.adobe.AdobeCreativeCloud.plist
```

恢复
```bash
launchctl load -w /Library/LaunchAgents/com.adobe.AdobeCreativeCloud.plist
```
