---
title: iOS WKWebview/UIWebview长按网页里的图片保存会闪退的问题
author: 独孤流
date: 2019-10-31 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

`webview save image crash`
iOS WKWebview/UIWebview长按网页里的图片保存会闪退的问题
解决方案：
在Info.plist的文件里添加使用照片的权限：
`NSPhotoLibraryAddUsageDescription`
效果如下：
![image](/assets/img/webview/webviewCrash1.webp)
参考：https://jira.appcelerator.org/browse/MOD-2398

不建议的一种解决方式：
在使用 WKWebView 展示 H5 时，如果 H5 中有图片，长按图片会出现弹框，在 iOS11 系统中，存储图像，如果未开启相册权限，会直接 Crash 掉：

解决方案一（原生解决）：　　
在代理方法中添加如下代码，禁掉弹框：

```
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    NSLog(@"webViewDidFinishLoad");
    [self.webView evaluateJavaScript:@"document.documentElement.style.webkitTouchCallout='none';" completionHandler:nil];
    [self.webView evaluateJavaScript:@"document.documentElement.style.webkitUserSelect='none';"completionHandler:nil];
}
```
解决方案二（H5）解决：
```
img { pointer-events: none; }
```

参考：[WebView 中图片长按出现弹框，点击存储图像闪退的解决方案](https://www.cnblogs.com/ZachRobin/p/9104118.html)