---
title: Linux选择默认编辑器
date: 2018-05-12 00:00:00
tags: ["Linux"]
abbrlink: linux-xuan-ze-mo-ren-bian-ji-qi
img: ""
comments: false
---

```
sudo update-alternatives --config editor 
```

运行命令后有类似下面的提示：
有 4 个候选项可用于替换 `editor` (提供 /usr/bin/editor)。

```
  选择       路径              优先级  状态
------------------------------------------------------------
* 0            /bin/nano            40        自动模式
  1            /bin/ed             -100       手动模式
  2            /bin/nano            40        手动模式
  3            /usr/bin/vim.basic   30        手动模式
  4            /usr/bin/vim.tiny    10        手动模式
```

要维持当前值[*]请按回车键，或者键入选择的编号
