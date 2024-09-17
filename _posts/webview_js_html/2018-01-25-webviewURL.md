---
title: iOS WebView打开URL时会对地址自动进行URL
author: 独孤流
date: 2018-01-25 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---
> 关键词： WebView、URLEnocde、loadRequest、loadHTMLString

### 一、现状
在一个客户端打开服务端传来的url，使用WebView打开时出现地址打不开的情况，见过比对检查，发现`loadRequest:`的地址和`- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType`中的获取的地址不一致，具体内容就是请求时会自动URLEncode
比如，
1、服务端的原地址为：`https://www.baidu.com?name=+kb`
2、服务端URLEncode后传给客户端的地址为：`https://www.baidu.com?name=%2bkb`
3、客户端用W`ebView的loadRequest:`加载后再拦截处`shouldStartLoadWithRequest: navigationType:`看到的却是`https://www.baidu.com?name=%252bkb`，也就是把服务端encode过的地址又重新encode了一遍，导致webView请求后会导致地址错误或参数错误的情况

### 二、解决方案
使用webview加载一段html，然后在html里跳转到对应的服务端encode过的URL地址
```
NSString *urlFromServer = @"https://www.baidu.com?name=%2bkb";
NSString *urlHtml = [NSString stringWithFormat:@"<script>window.location=\"%@\";</script>",urlFromServer];
[webview loadHTMLString:urlHtml baseURL:nil];
```