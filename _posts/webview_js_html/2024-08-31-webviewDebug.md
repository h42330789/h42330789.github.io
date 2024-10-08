---
title: 使用Safari调试真机或模拟器的wkwebview
author: 独孤流
date: 2024-08-30 01:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

参考：
- [iOS16.4 WKWebview 不能用Safari调试](https://cloud.tencent.com/developer/article/2264775)
- [iOS 16.4后 Safari 开发中不能调试Web页面](https://blog.csdn.net/siwen1990/article/details/130363477)
- [WKWebView 使用safari在真机上调试网页](https://juejin.cn/post/7125694000689840135)

>Xcode 升级到 14.3后，模拟器是iOS 16.4 的，想通过Safari -> Developer -> Web Inspector查看，结果发现查看不了,需要设置isInspectable
官方文档 isInspectable | Apple Developer Documentation
简单的说，在iOS 16.4 WKWebView 增加了一个属性 isInspectable 用来控制 => 是否可以使用Safari 检查网页中的JS Code, HTML 等内容, 需要手动设置.

```

// OC
if (@available(iOS 16.4, *)) {
    webView.inspectable = true;
} else {
    // Fallback on earlier versions
}

// Swift
if #available(iOS 16.4,*) {
    webView.isInspectable = true
} else {
    // Fallback on earlier versions
}
```