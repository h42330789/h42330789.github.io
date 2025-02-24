---
title: iOS18模拟器UIImagePicker位置异常
author: 独孤流
date: 2025-02-22 00:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

xcode升级后发现在之前模拟器运行选择图片的`UIImagePicker`展示异常，点击没有任何效果，但是使用一个新的空白项目又是正常的。经过对比配置，发现是`Excluded Archtectures`->`Debug`->`Any iOS Simulator SDK`配置了`arm64`,只要配置了这个就会导致在iOS18以上的模拟器上出现这个问题，后面研究了些资料，发现这个是苹果官方就有的问题

![image](/assets/img/other/ios18_picker.png)
![image](/assets/img/other/ios18_picker2.png)

参考；
https://developer.apple.com/forums/thread/765079
https://github.com/flutter/flutter/issues/155401