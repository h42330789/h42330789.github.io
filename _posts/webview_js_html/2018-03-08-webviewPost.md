---
title: iOS WebView使用POST方式加载URL及传参
author: 独孤流
date: 2018-03-08 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

> 在APP的使用场景中有一种是后端传一个URL给客户端，然后客户端使用webview打开这个URL，但一般默认webview加载URL都是使用get请求，但是某些特殊的请求会要求是POST方式的，这个问题解决方式如下：

参考：

- [HTML中如何用post提交数据](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.php.cn%2Fdiv-tutorial-400681.html)
- [四种常见的 POST 提交数据方式 专题](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fsoftidea%2Fp%2F5745369.html)


#### 方法一、让服务器接口一并返回要打开的URL、method(post)、参数，让后webview发起请求时使用NSMutableRequest创建request请求，代码如下：
```
- (BOOL)isEmptyString:(NSString *)string
{
    if (string == nil || [string isKindOfClass:[NSNull class]] || [string isEqualToString:@"null"] ||
        [string isEqualToString:@"(null)"] || [string isEqualToString:@""] ||
        [string isEqualToString:@" "]) {
        return YES;
    }
    if ([[string stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]] length] ==
        0) {
        return YES;
    }
    return NO;
}
- (void)loadUrl:(NSString *)urlStr method:(NSString *)method params:(NSDictionary *)params{
        NSURL *url = [NSURL URLWithString:urlStr];
        NSMutableURLRequest *requestM = [NSMutableURLRequest requestWithURL:url];
        if (method != nil && [method.uppercaseString isEqualToString:@"POST"]) {
            // 如果有webMethod并且是POST,则POST方式组合提交
            [requestM setHTTPMethod:@"POST"];
            NSString *body = nil;
            for (NSString *key in params.allKeys) {
                if ([self isEmptyString: body]) {
                    body = [NSString stringWithFormat:@"%@=%@",key,params[key]];
                }else{
                    body = [NSString stringWithFormat:@"%@&%@=%@",body,key,params[key]];
                }
            }
            [requestM setHTTPBody: [body dataUsingEncoding: NSUTF8StringEncoding]];
        }
      
        [self.webView loadRequest:requestM];
}

```
#### 方法二、使用webview加载html字符串执行form表单POST提交，html字符串可以是后台拼接好传给客户端执行或在客户端拼接也可以
```
- (void)loadHtmlWithUrl:(NSString *)urlStr method:(NSString *)method params:(NSDictionary *)params{
        NSString *htmlText = nil;
       if (method != nil && [method.uppercaseString isEqualToString:@"POST"]) {
          // 拼接form表单的请求地址
            htmlText = [NSString stringWithFormat:@"<html><body><form id=\"login\" name=\"login\" action=\"%@\" method=\"post\">",urlStr];
            // 拼接form表单的请求参数
            for (NSString *key in params.allKeys) {
                 htmlText = [NSString stringWithFormat:@"%@<input type=\"hidden\" name=\"%@\" value=\"%@\" />",htmlText,key,params[key]];
            }
           // 拼接form表单的尾部及JavaScript自动执行form表单提交
            htmlText = [NSString stringWithFormat:@"%@</form></body><script>document.login.submit();</script></html>;",htmlText];
       }

    [self.webView loadHTMLString:htmlText baseURL:nil];
}
```
拼接后的完整格式如下：
```
<html> 
<body>
     <form id="login" name="login" action="https://www.baidu.com" method="post">
        <input type="hidden" name="name" value="aaa" />
        <input type="hidden" name="age" value="123"/>
    </form>
</body>
<script>
    document.login.submit();
</script> 
</html>
```