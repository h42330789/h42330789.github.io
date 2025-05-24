---
title: 在高版本的Mac系列上安装使用低版本Xcode
author: 独孤流
date: 2025-05-24 01:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

> ### 前言
> 最近由于有用户反馈在一些低版本系统如iOS14上出现一些内嵌webview展示异常，但是由于我们自己的测试设备已经没有这么低的系统，Xcode可以安装的模拟器系统也没有这么低，只能安装低版本的Xcode，而Mac系统升级后低版本的Xcode安装后也打不开，特解决次问题

研究了安装旧版本xcode、安装低版本模拟器都没法在高版本直接运行iOS14的模拟器

#### 解决方案一、在高版本的Mac系统上下载低版本Xcode安装
参考：
[[iOS]高版本MacOS运行低版本Xcode](https://blog.csdn.net/qq_38973710/article/details/136637724)

Xcode下载地址：
- https://developer.apple.com/download/more/
- https://developer.apple.com/download/all/
- https://developer.apple.com/download/all/?q=xcode

1、将手动下载的低版本Xcode解压后放入到应用文件夹里
![image](/assets/img/other/lowXcode.png)

2、关闭之前测的xcode

3、运行命令
```
/Applications/Xcode_14.2.app/Contents/MacOS/Xcode ; exit;
```

方案二：
找到`/Applications/Xcode_14.2.app/Contents/Info.plist`
搜索`CFBundleVersion`,找到`Bundle version`的值，我的`xcode14.2`的值为`21534`
![image](/assets/img/other/xcodeBuildVersion.png)
```
mv /Applications/Xcode_14.2.app ~/Downloads
# 修改Xcode的buildVersion
# /Applications/Xcode_14.2.app/Contents/Info.plist
mv ~/Downloads/Xcode_14.2.app /Applications 
```
Sequoia 15.3.2，已经不允许安装Xcode15
替换为`22256`， 因为是`22256`是从`Xcode15.4`开始的
Sequoia 15.3.2，已经可以安装Xcode15
替换为`23051`， 因为是`23051`是从`Xcode16.0`开始的


#### 解决方案二、在高版本的Xcode安装低版本的模拟器

~~该路已行不通，直接安装不了~~~

iOS 14.3
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK14_3-14.3.1.1611873653.dmg

iOS 14.2
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK14_2-14.2.1.1605311653.dmg

iOS 14.1
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK14_1-14.1.1.1604100028.dmg

iOS 14.0
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK14_0-14.0.1.1604100028.dmg

iOS 13.7
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_7-13.7.1.1599165590.dmg

iOS 13.6
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_6-13.6.1.1597276955.dmg

iOS 13.5
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_5-13.5.1.1591226335.dmg

iOS 13.4
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_4-13.4.1.1586370836.dmg

iOS 13.3
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_3-13.3.1.1580170331.dmg

iOS 13.2
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_2-13.2.1.1575590084.dmg

iOS 13.1
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_1-13.1.1.1571440502.dmg

iOS 13.0
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK13_0-13.0.1.1571440502.dmg

iOS 12.4
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK12_4-12.4.1.1568665771.dmg

iOS 12.2
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK12_2-12.2.1.1557987768.dmg

iOS 12.1
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK12_1-12.1.1.1543439531.dmg

iOS 12.0
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK12_0-12.0.1.1537588161.dmg

iOS 11.4
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK11_4-11.4.1.1527703358.dmg

iOS 11.3
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK11_3-11.3.1.1524350608.dmg

iOS 11.2
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK11_2-11.2.1.1516308624.dmg

iOS 11.1
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK11_1-11.1.1.1510784422.dmg

iOS 11.0
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK11_0-11.0.1.1508875951.dmg

iOS 10.3.1
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK10_3-10.3.1.1495751597.dmg

iOS 10.2
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK10_2-10.2.1.1484185528.dmg

iOS 10.1
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK10_1-10.1.1.1476902849.dmg

iOS 10.0
- https://devimages-cdn.apple.com/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK10_0-10.0.1.1474488730.dmg

iOS 9.3
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK9_3-9.3.1.1460411551.dmg

iOS 9.2
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK9_2-9.2.1.1451951473.dmg

iOS 9.1
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK9_1-9.1.1.1446593668.dmg

iOS 9.0
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK9_0-9.0.1.1443554484.dmg

iOS 8.4
https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK8_4-8.4.1.1435785476.dmg

iOS 8.3
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK8_3-8.3.1.1434581536.dmg

iOS 8.2
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK8_2-8.2.1.1434581536.dmg

iOS 8.1
- https://devimages.apple.com.edgekey.net/downloads/xcode/simulators/com.apple.pkg.iPhoneSimulatorSDK8_1-8.1.1.1434581536.dmg