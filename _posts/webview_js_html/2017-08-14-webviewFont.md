---
title: iOS设置WebView中的字体及型号
author: 独孤流
date: 2017-08-14 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

> 近期产品提出要将APP中h5显示的网页正文字体及大小要和APP原生的一致，也就是要修改网页里正文部分的字体名称和大小，由于使用的字体是系统自带的字体，所以直接在webViewDidFinishLoad:方法中对对应的节点设置一下字体就可以了，具体代码如下：

```
- (void)webViewDidFinishLoad:(UIWebView *)webView
{
    // 设置字体
    NSString *fontFamilyStr = @"document.getElementsByTagName('body')[0].style.fontFamily='PingFangSC-Light';";
    [webView stringByEvaluatingJavaScriptFromString:fontFamilyStr];
    // 设置字体大小
    NSString *fontSizeStr = @"document.getElementsByTagName('section')[0].style.fontSize='14px';";
    [webView stringByEvaluatingJavaScriptFromString:fontSizeStr];
}
```

如果使用的字体不是系统自带的字体，而是加载第三方的字体，找到了一篇博客供参考：
[UIWebView使用app内自定义字体](https://www.jianshu.com/p/ac319f4c5daa)