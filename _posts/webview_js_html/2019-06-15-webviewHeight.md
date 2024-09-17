---
title: iOS WKWebView、UIWebView获取网页内容高度
author: 独孤流
date: 2018-11-09 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

参考：

- [iOS WKWebView的使用](https://www.jianshu.com/p/5cf0d241ae12)
- [精准获取webView内容高度]()

> 项目UI需要，页面部分内容是网页富文本或者加载的是网页，然后展示的UI高度还跟展示的网页内容高度相关

 一般可以拿到的有：FitSize offset scrollView.contentSize

一、加载富文本
1.1 WKWebView
```
  self.webView = [[WKWebView alloc] initWithFrame:CGRectZero];
 [self.view addSubView:self.webView];

NSString *stringContent = @"xxxx";
// 默认设置文本字体
  NSString *htmlTxt = [NSString stringWithFormat:@"<html><body style='font-size:26pt;'>%@</body></html>",stringContent ?: @""];
  [self.webView loadHTMLString:htmlTxt baseURL:nil];
  WS(weakSelf)
// 隔一段时间再取高度，好让WebView渲染完取的高度更准确
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            
            NSString *jsTxt = @"document.body.offsetHeight;";
            [weakSelf.webView evaluateJavaScript:jsTxt completionHandler:^(id val, NSError *error) {
                if(val != nil){
                    CGSize fittingSize = [weakSelf.webView sizeThatFits:CGSizeZero];
                    CGFloat offsetHeight = [val floatValue];
                    CGFloat contentHeight = weakSelf.webView.scrollView.contentSize.height;
                    NSLog(@"%f %f %f",fittingSize.height,offsetHeight,contentHeight);
                    CGFloat boxHeight = contentHeight + 50;
                    // 防止过长，让web内容自己滚动
                    if(boxHeight>(Main_Screen_Height-30)){
                        boxHeight = Main_Screen_Height-30;
                    }
                   // 设置展示的UI相关高度
                    xxxxx
                }
            }];
 });
```
1.2 UIWebView
```
self.webView = [[WKWebView alloc] initWithFrame:CGRectZero];
 [self.view addSubView:self.webView];

NSString *stringContent = @"xxxx";
// 默认设置文本字体
  NSString *htmlTxt = [NSString stringWithFormat:@"<html><body style='font-size:26pt;'>%@</body></html>",stringContent ?: @""];
  [self.webView loadHTMLString:htmlTxt baseURL:nil];
  WS(weakSelf)
// 隔一段时间再取高度，好让WebView渲染完取的高度更准确
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            
    NSString *jsTxt = @"document.body.offsetHeight;";
    CGFloat offsetHeight = [[weakSelf.webView stringByEvaluatingJavaScriptFromString:jsTxt] floatValue];
    CGSize fittingSize = [weakSelf.webView sizeThatFits:CGSizeZero];
                 
    CGFloat contentHeight = weakSelf.webView.scrollView.contentSize.height;
    NSLog(@"%f %f %f",fittingSize.height,offsetHeight,contentHeight);
 });
```
二、加载网页
和富文本一样，只需要在对应的
`UIWebView`的代理方法
`- (void)webViewDidFinishLoad:(UIWebView *)webView`
或WKWebView的代理方法处理即可
`- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation`
