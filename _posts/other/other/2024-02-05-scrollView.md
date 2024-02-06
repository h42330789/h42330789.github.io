---
title: 模拟器UIScrollView滑动卡顿及不能直接使用LookIn的断点接
author: 独孤流
date: 2024-01-27 01:04:00 +0800
categories: [other, 其他]
tags: [UIScrollView]     # TAG names should always be lowercase
---

> ### 前言
> 在接手的一旧项目，在模拟器运行后发现所有的ScrollView的滑动都没有了惯性，滑动很重，但是在真机上运行确是正常的，同时使用Xcode创建新项目时又是正常的

1、滑动卡顿,效果如下
![滑动卡顿](/assets/img/other/other-scroll1.mp4)

解决方案：\
`Build Settings` -> `Architectures` -> `Excluded Architectures` -> `Debug`，删掉`arm64`的配置
![arm64](/assets/img/other/other-other_scroll_arm64_1.png)
![arm64](/assets/img/other/other-other_scroll_arm64_2.png)
![arm64](/assets/img/other/other-other_scroll_arm64_3.png)

修改后的模拟器就运行很顺滑有惯性了
![滑动卡顿](/assets/img/other/other-scroll2.mp4)

----
在之前的文章里直接使用断点引入就可以进行UI的debug，但是接手的项目一直报错：\
```
Warning: Error creating LLDB target at path '/Users/xxx/Library/Developer/Xcode/DerivedData/TestTable-bjhefqpmxnkykgapypdspvdmdtph/Build/Products/Debug-iphonesimulator/TestTable.app'- using an empty LLDB target which can cause slow memory reads from remote devices.
```
解决方案：同上\
`Build Settings` -> `Architectures` -> `Excluded Architectures` -> `Debug`，删掉`arm64`的配置

[使用最新的LookIn进行断点调试查看UI](https://www.jianshu.com/p/3ee8960c1ffb)

参考：\
[M1适配,New Build System,模拟器卡顿,Rosetta](https://www.jianshu.com/p/9df49126ec27)\
[Using an empty LLDB target which can cause slow memory...](https://stackoverflow.com/questions/64114768/using-an-empty-lldb-target-which-can-cause-slow-memory-reads-from-remote-devices)

----
如果引入了某些SDK，比如声网`AgoraRtcEngine_iOS_Crypto`,在M系列芯片的模拟器上运行，又会报错
> ld: building for 'iOS-simulator', but linking in dylib (/Users/xx/xx/TestTable/Pods/AgoraRtcEngine_iOS_Crypto/Agora_Native_SDK_for_iOS_FULL/libs/AgoraRtcCryptoLoader.framework/AgoraRtcCryptoLoader) built for 'iOS'
clang: error: linker command failed with exit code 1 (use -v to see invocation)

解决方案，必须把`Build Settings` -> `Architectures` -> `Excluded Architectures` -> `Debug`，增加`arm64`的配置


参考：\
[整理一下脑子里混乱的Architectures](https://juejin.cn/post/7034782069410496542)\
[building for iOS Simulator, but linking in object file built for iOS](https://www.jianshu.com/p/0fc2a6402870)\
[报错building for iOS Simulator, but linking in object file built for iOS](https://blog.csdn.net/u011224726/article/details/124453819)\
[Xcode building for iOS Simulator, but linking in an object file built for iOS, for architecture 'arm64'](https://stackoverflow.com/questions/63607158/xcode-building-for-ios-simulator-but-linking-in-an-object-file-built-for-ios-f)