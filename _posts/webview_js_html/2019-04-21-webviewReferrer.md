---
title: WebView构造中间页自由设置Referrer
author: 独孤流
date: 2019-04-21 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

> 三、伪造Referrer、增加中间页空白跳转
业务需求：在接入一个第三方支付时，基本流程是生产一个订单，然后后端返回一个URL用浏览器打开，之后就是打开原生的微信或支付宝支付，但其中一家支付厂商的支付URL有个特殊的要求，就是在浏览器发起请求时要设置Referrer这个请求头，但当前这个请求本身是第一次请求，浏览器默认是的referrer事空的，必须要在客户端自己想办法加上。

请求Referrer的地址，使用NSURLProtocol拦截WebView的这个请求并构造一个本地数据中转，通过这个中转再跳转到目标地址，代码如下：

### 亲测使用有效

目标地址：https://www.baidu.com 要伪造的referrer地址：https://www.google.com
这两个地址在实际开发使用中需要根据需要做成参数动态使用

1、创建URLProtocol的子类
LHJumpProtocol.h
```
#import <Foundation/Foundation.h>
@interface LHJumpProtocol : NSURLProtocol
@end
```

LHJumpProtocol.m
```
#import "LHJumpProtocol.h"
static NSString * const URLProtocolHandledKey = @"URLProtocolHandledKey_jump";

@interface LHJumpProtocol()<NSURLConnectionDelegate>
@property (nonatomic, strong) NSURLConnection *connection;
@end
@implementation LHJumpProtocol
/*
 这个方法是决定这个 protocol 是否可以处理传入的 request 的如是返回 true 就代表可以处理,如果返回 false 那么就不处理这个 request 。
 */
+ (BOOL)canInitWithRequest:(NSURLRequest *)request
{
    //看看是否已经处理过了，防止无限循环
    if ([NSURLProtocol propertyForKey:URLProtocolHandledKey inRequest:request]) {
        return NO;
    }
    
    NSString *url = request.URL.absoluteString;
    // 只有请求的是中转的地址才处理
    if ([url isEqualToString:@"https://www.google.com"]) {
        return YES;
    }
   return NO;
}
   
/*
 这个方法主要是用来返回格式化好的request，如果自己没有特殊需求的话，直接返回当前的request就好了。如果你想做些其他的，比如地址重定向，或者请求头的重新设置，你可以copy下这个request然后进行设置
 */
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request
{
    NSMutableURLRequest *mutableReqeust = [request mutableCopy];
   // xxx 可以在这里对request进行相关的设置参数，重定向等处理
    return mutableReqeust;
    
}
/**
 该方法主要是判断两个请求是否为同一个请求，如果为同一个请求那么就会使用缓存数据。通常都是调用父类的该方法
 */
+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b
{
    return [super requestIsCacheEquivalent:a toRequest:b];
}
- (void)startLoading
{
    
    NSMutableURLRequest *mutableReqeust = [[self request] mutableCopy];
    //打标签，防止无限循环
    [NSURLProtocol setProperty:@YES forKey:URLProtocolHandledKey inRequest:mutableReqeust];
    
   // 虚拟一般网络请求时服务器返回的中转网页内容，这段网页数据的作用是告诉浏览器请求都带上referrer，并且会加载这段网页后自动跳转到目标地址去
 NSString *targetUrl = @"http://www.baidu.com";
 NSString *text = [NSString stringWithFormat:@"<html> <META content=\"always\" name=\"referrer\"><script>try{if(window.opener&&window.opener.bds&&window.opener.bds.pdc&&window.opener.bds.pdc.sendLinkLog){window.opener.bds.pdc.sendLinkLog();}}catch(e) {};window.location.replace(\"%@\")</script> <noscript><META http-equiv=\"refresh\" content=\"0;URL='%@'\"></noscript></META></html>",targetUrl,targetUrl];
    
// 将字符串转换成NSData数据
 NSData *data = [text dataUsingEncoding:NSUTF8StringEncoding];
 // 将数据构造成返回的response
 NSURLResponse *response = [[NSURLResponse alloc] initWithURL:mutableReqeust.URL MIMEType:@"text/html" expectedContentLength:data.length textEncodingName:nil];
    
    [self.client URLProtocol:self
          didReceiveResponse:response
          cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    
    [self.client URLProtocol:self didLoadData:data];
    [self.client URLProtocolDidFinishLoading:self];
    
}
- (void)stopLoading
{
    [self.connection cancel];
}
#pragma mark - NSURLConnectionDelegate

- (void) connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
}

- (void) connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    [self.client URLProtocol:self didLoadData:data];
}

- (void) connectionDidFinishLoading:(NSURLConnection *)connection {
    [self.client URLProtocolDidFinishLoading:self];
}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}
```

2、在AppDelegate里注册protocol
```
#import "LHAppDelegate.h"
#import "LHJumpProtocol.h"
@interface LHAppDelegate ()
@end
@implementation LHAppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
// xxxx 其他启动处理
// 注册protocol
[NSURLProtocol registerClass:[LHJumpProtocol class]];
    return YES;
}
@end
```
3、使用UIWebView发起请求
```
UIWebView *webView = [ [UIWebView alloc] iniwWithFrame:self.view.bounds];
// 这里请求的要是需要的referrer的地址，这样才能构造一个虚拟的referrer
NSURL *url = [NSURL URLWithString:@"http://www.google.com"];
NSURLRequest *request = [NSURLRequest requestWithURL:url]; [webView loadRequest:request];
```