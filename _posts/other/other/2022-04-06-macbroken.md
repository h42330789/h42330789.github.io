---
title: Mac安装软件的“已损坏，无法打开"
author: 独孤流
date: 2022-04-06 23:12:00 +0800
categories: [other, 其他]
tags: [mac]     # TAG names should always be lowercase
---
参考：
- [最新｜解决Mac安装软件的“已损坏，无法打开。 您应该将它移到废纸篓”问题](https://zhuanlan.zhihu.com/p/135948430)
- [macOS 10.13 安装允许任何来源没了怎么开启](https://zhuanlan.zhihu.com/p/51328476)


### 一、已损坏，无法打开。 您应该将它移到废纸篓
让允许任何来源的app可以使用

![image](/assets/img/other/macbroken_01.webp)

如果没有`任何来源`选项，在终端terminal里输入如下命令，然后退出设置，重新进入设置就好了
```
sudo spctl  --master-disable
```

### 二、应用程序“xxxx”无法打开
最新的macOS 10.15及更高的版本，允许运行
```
sudo xattr -r -d com.apple.quarantine /Applications/xxxx.app
```