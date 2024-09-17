---
title: 使用wkwebview打开页面会被提示不是Safari的及自定义UserAgent问题
author: 独孤流
date: 2024-09-17 02:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

参考：
- [WKWebView iOS17 设置 UserAgent_ios17 wkwebview useragent-CSDN 博客](https://blog.csdn.net/xo19882011/article/details/134031247)
- [WKWebView 设置自定义UserAgent正确姿势](https://juejin.cn/post/6844903632152821773)

> App内使用WKWebview，有些h5会根据`UserAgent`判断是在Safari里还是APP内，在APP内会提示进入Safari，体验不好，可以在在APP里设置特定的内容干扰判断提高体验

customUserAgent > UserDefault > applicationNameForUserAgent
```

class TestVC: UIViewController {
    var webview: WKWebView = {
        let config = WKWebViewConfiguration()
        // 方式一、设置applicationNameForUserAgent
        // config.applicationNameForUserAgent = "Mozilla/5.0 (iPhone; CPU iPhone OS 17_4 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.4 Mobile/15E148 Safari/604.1"
        let v = WKWebView(frame: .zero, configuration: config)
        if #available(iOS 16.4,*) {
            v.isInspectable = true
        } else {
            // Fallback on earlier versions
        }
        // 方式二、设置customUserAgent
        let oldUserAgent = v.value(forKey: "userAgent") as? String ?? ""
        v.customUserAgent = "\(oldUserAgent) Safari/604.1"
        return v
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.addSubview(webView)
        if let url = URL(string: "https://wwww.google.com") {
            self.webView.load(URLRequest(url: url))
        }
    }
}
```