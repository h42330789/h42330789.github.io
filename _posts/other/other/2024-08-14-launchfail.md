---
title: Xcode运行模拟器失败
author: 独孤流
date: 2024-08-14 01:04:00 +0800
categories: [other, 其他]
tags: [other]     # TAG names should always be lowercase
---

> `Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding`

参考：
["Failed to start launchd_sim: could not bind to session, launchd_sim may have crashed or quit responding"](https://stackoverflow.com/questions/65172944/when-running-on-older-ios-simulator-error-failed-to-start-launchd-sim-could-n)
![image](/assets/img/other/launchFail1.png)
解决方案：
```
# 删除受影响的缓存
rm -rf ~/Library/Developer/CoreSimulator/Caches
```