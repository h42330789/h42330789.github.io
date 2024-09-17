---
title: iOS UIWebView 与WKWebView集锦
author: 独孤流
date: 2018-11-09 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

- [使用WKWebView进行性能调优](https://xibhe.com/2018/02/03/WKWebView-disabuse/index.html)
- [WebView性能、体验分析与优化](https://tech.meituan.com/WebViewPerf.html)
- [UIWebView、WKWebView使用详解及性能分析](https://www.cnblogs.com/junhuawang/p/5759224.html)
- [webView:decidePolicyForNavigationAction:decisionHandler:](https://developer.apple.com/documentation/webkit/wknavigationdelegate/1455641-webview?language=objc)
- [WKWebView 那些坑](https://mp.weixin.qq.com/s/rhYKLIbXOsUJC_n6dt9UfA)
- [iOS WebView 中的 Cookie 处理业务场景“IP直连”方案说明](https://helpcdn.aliyun.com/knowledge_detail/60199.html)
- [Developer wknavigationdelegate documentation](https://developer.apple.com/documentation/webkit/wknavigationdelegate/1455641-webview?language=objc)
- [WKWebView and UIWebView Cookie](https://forums.developer.apple.com/thread/50057)
- [iOS H5容器的一些探究（一）：UIWebView和WKWebView的比较和选择](https://www.jianshu.com/p/84a6b1ac974a)
- [iOS H5容器的一些探究（二）：iOS下的黑魔法NSURLProtocol](https://www.jianshu.com/p/03ddcfe5ebd7)
- [WKWebView和UIWebView对比](https://www.jianshu.com/p/79e329ff8953)
- [使用WKWebView替换UIWebView](https://www.jianshu.com/p/6ba2507445e4)

WKWebView 坑
WKWebView 白屏问题
WKWevView Cookie 问题
WKWebView NSURLProtocol问题
WKWebView 页面样式问题
WKWebView 截屏问题
WkWebView crash 问题
WKWebView和UIWebView 的区别
WKWebView 更快（占用内存可能只有 UIWebView 的1/3~1/4），没有缓存，更为细致地拆分了 UIWebViewDelegate 中的方法。