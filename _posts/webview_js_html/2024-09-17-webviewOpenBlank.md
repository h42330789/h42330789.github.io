---
title: 使用wkwebview打开页面内使用blank打开新页面没反应问题及直接下载APP
author: 独孤流
date: 2024-09-17 01:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---


> 经常APP内嵌的h5打卡时，写h5的人习惯使用新tab打开页面，但是APP默认是不支持，需要特别适配才支持
> 同时一些APP内嵌网页希望可以点击某个链接就可以下载企业签的APP或调整到Appstore去下载APP

```

class TestVC: UIViewController {
    var webView = WKWebView()
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.addSubview(webView)
        if let url = URL(string: "https://wwww.google.com") {
            self.webView.load(URLRequest(url: url))
        }
    }
    func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
        
        if let rtn = self.delegate?.webview(self, shouldStartLoadWithRequest: navigationAction.request, navigationType: navigationAction.navigationType) {
            if let url = navigationAction.request.url, isInstallUrl(str: navigationAction.request.url?.absoluteString) {
                // 识别出是安装链接时，只掉调用系统方法打开安装链接
                DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(10), execute: {
                    if UIApplication.shared.canOpenURL(url) {
                        UIApplication.shared.open(url, options: [:]) { result in
    //                        print(">>> url: \(url) result: \(result)")
                        }
                    } else {
    //                    print(">>> url: \(url) 打不开")
                    }
                })
                decisionHandler(.cancel)
            } else {
                decisionHandler(.allow)
                
            }
        } else {
            decisionHandler(.allow)
        }
        
    }
    
    // 如果是使用_blank的方式打开新页面，APP拦截处理成在同一个窗口打开页面
    func webView(_ webView: WKWebView, createWebViewWith configuration: WKWebViewConfiguration, for navigationAction: WKNavigationAction, windowFeatures: WKWindowFeatures) -> WKWebView? {
        if let urlStr = navigationAction.request.url?.absoluteString, urlStr.isNotBlank {
            if navigationAction.targetFrame == nil || navigationAction.targetFrame?.isMainFrame == false {
                webView.load(navigationAction.request)
                // let scriptChangeHeight = "window.location.href=\"\(urlStr)\"; "
                webView.evaluateJavaScript(scriptChangeHeight)
            }
        }
        
        return nil
        
    }

    func isInstallUrl(str: String?) -> Bool {
        guard let str = str, str.count > 0 else {
            // 不是instal地址
            return false
        }
        // itms-services://?action=download-manifest&url=https://xx.xx.xx/xxx.plist
        let urlStr = str.lowercased()
        if urlStr.hasPrefix("itms-services://"),
           urlStr.hasSuffix(".plist"),
           urlStr.contains("action=download-manifest"),
           urlStr.contains("url=https"){
            return true
        }
        // appstore下载
        // https://apps.apple.com/jp/app/surge-5/id1442620678
        // https://apps.apple.com/cn/app/idxxxxxx
        if urlStr.hasPrefix("https://apps.apple.com"),
           urlStr.contains("app/"),
           urlStr.contains("/id"){
            return true
        }
        return false
    }
}
```