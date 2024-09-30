---
title: SDK does not contain 'libarclite'
author: 独孤流
date: 2024-09-30 01:04:00 +0800
categories: [other, 其他]
tags: [libarclite]     # TAG names should always be lowercase
---

参考：
- [SDK does not contain 'libarclite'](https://github.com/yuehuig/libarclite)

> `clang: error: SDK does not contain 'libarclite' at the path '/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/arc/libarclite_iphonesimulator.a'; try increasing the minimum deployment target`

拷贝整个 arc 目录到新Xcode对应目录下
```
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/
```
![image](/assets/img/other/libarclite.png)

----
