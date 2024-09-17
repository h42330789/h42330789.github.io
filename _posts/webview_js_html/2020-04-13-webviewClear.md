---
title: WKWebView/UIWebView速度优化及清空访问栈
author: 独孤流
date: 2020-04-13 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

> ### 前言
> 接到一个提升App里h5的加载速度优化的问题，参考了很多文章后决定从webview缓存池、并行加载开始

1、统计从的耗时
点击入口->controller创建(init)->设置访问的url地址->调用webview的loadRequest->webview回调start->webview回调finish

统计发现，在调用点击入口到loadRequest这个过程耗费了0.5s左右，
1、于是参考VasSonic在Controller初始化Init时就调用初始好Webview，后来改成在APP启动时就创建一个Webview的缓存池，这样在页面一开始时就已经有webkit内核初始化好的Webview
2、在设置好url后立马调用Webview的loadRequest

但是这个过程中遇到了几个问题，在使用Webview缓存池使用Webview时会有Webview留存上次访问栈的问题，同时在下次请求时还会先展示上次请求过的内容，具体解决办法如下：

> 清空Webview的缓存栈：执行js方法
```
[webView evaluateJavaScript@" window.location.replace( 'https://www.baidu.com' )"]
```

> 删除上次访问的内容
```
[webView evaluateJavaScript@"window.document.body.remove();"]
```

> 清空sessionStorage里的内容
```
[webView evaluateJavaScript@"window.sessionStorage.removeItem(\"token\");"]
```

> 清空locationStorage里的内容
```
[webView evaluateJavaScript@"window.localStorage.removeItem(\"token\");"]
```

> 清空cookie里的内容
```
[webView evaluateJavaScript:@"document.cookie = \"token=xxx;expires=Thu, 01 Jan 1970 00:00:01 GMT;path=/;\""]
```

完整处理分类：
`WKWebView+Test.h`
```
#import <WebKit/WebKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface WKWebView (Test)
// 删除webview里的内容
- (void)test_clearBody;
/// 替换url请求，清空访问栈
- (void)test_replaceUrl:(NSString *)url;
/// 清除LocalStorage里的内容
- (void)test_deleLocalStorage:(NSString *)key;
/// 清除SessionStorage里的内容
- (void)test_deleSessionStorage:(NSString *)key;
/// 清除cookie里的内容
- (void)test_deleCookie:(NSString *)key path:(NSString *)path;
@end
NS_ASSUME_NONNULL_END
```
`WKWebView+Test.m`
```
#import "WKWebView+Test.h"

@implementation WKWebView (Test)
// 删除webview里的内容
- (void)test_clearBody{
   [self evaluateJavaScript@"window.document.body.remove();"]
}
/// 替换url请求，清空访问栈
- (void)test_replaceUrl:(NSString *)url {

    if ([url.lowercaseString hasPrefix:@"http"]) {
        NSString *href = [NSString stringWithFormat:@"window.location.replace('%@')",url];
        [self evaluateJavaScript:href completionHandler:nil];
    }
}
/// 清除LocalStorage里的内容
- (void)test_deleLocalStorage:(NSString *)key {
    NSString *text = [NSString stringWithFormat:@"window.localStorage.removeItem(\"%@\",\"\");",key];
    [self evaluateJavaScript:text completionHandler:nil];
}
/// 清除SessionStorage里的内容
- (void)test_deleSessionStorage:(NSString *)key {
    NSString *text = [NSString stringWithFormat:@"window.sessionStorage.removeItem(\"%@\",\"\");",key];
    [self evaluateJavaScript:text completionHandler:nil];
}
/// 清除cookie里的内容
- (void)test_deleCookie:(NSString *)key path:(NSString *)path {
    NSString *text = [NSString stringWithFormat:@"document.cookie = \"%@=empty;expires=Thu, 01 Jan 1970 00:00:01 GMT;path=%@;\"",key,path];
    [self evaluateJavaScript:text completionHandler:nil];
}
@end
```
> 针对`WKWebview`请求网络时cookie丢失的问题，对重定向的请求都重新将cookie读取出来放入到请求的header的cookie里
```
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    [self redirectRequest: navigationAction.request];
    decisionHandler(WKNavigationActionPolicyAllow);
}

- (NSURLRequest *)redirectRequest:(NSURLRequest *)request
{
    NSMutableURLRequest *redirectRequest;
    if ([request isKindOfClass:[NSMutableURLRequest class]]) {
        redirectRequest = (NSMutableURLRequest *)request;
    } else {
        redirectRequest = [request mutableCopy];
    }
NSArray<NSHTTPCookie *> *cookies = [NSHTTPCookieStorage sharedHTTPCookieStorage].cookies;
    NSDictionary *cookieHeaders = [NSHTTPCookie requestHeaderFieldsWithCookies:cookies];
    if (originCookies.count) {
        NSMutableDictionary *redirectHeaders = request.allHTTPHeaderFields.mutableCopy;
        [redirectHeaders setValuesForKeysWithDictionary:cookieHeaders];
        redirectRequest.allHTTPHeaderFields = redirectHeaders;
    }
    return redirectRequest;
}
```
延伸阅读：

[苹果完全禁用第三方Cookie，就能保护好用户隐私了吗？](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.huxiu.com%2Farticle%2F347023.html)
[Cookie写入之path的坑](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2F52liming%2Fp%2F9536805.html)