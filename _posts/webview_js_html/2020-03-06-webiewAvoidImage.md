---
title: WKWebView拦截网页里请求的图片、JS/CSS资源
author: 独孤流
date: 2020-03-06 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

> ### 前言
> 接到一个需求，需要拦截APP内嵌的的webview里的h5的请求，包含普通的地址请求，还要拦截图片、JS、CSS等资源请求，找了好几个方案，发现在处理www.baidu.com时都很好的匹配，一旦患上我们自己的地址后就各种白屏或网络请求错误，或者拦截不到图片和资源，终于在GitHub找到了一个demo，使用注入的方式很好的实现的这个需求

demo Git地址：[WKWebViewRequestHook](https://github.com%2Ffenglee594%2FWKWebViewRequestHook)
另外发现一个很适合学习研究的demo [MyBrowser](https://github.com/luowei/MyBrowser)

一、拦截一般的通过document.url的方式打开的地址
```
// 根据WebView对于即将跳转的HTTP请求头信息和相关信息来决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
   // 处理拦截逻辑
 //decisionHandler(WKNavigationActionPolicyCancel);
// decisionHandler(WKNavigationActionPolicyAllow);
}
```

二、拦截图片、JS、CSS资源
参考demo：[WKWebViewRequestHook](https://github.com/fenglee594/WKWebViewRequestHook)