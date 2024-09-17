---
title: 使用Webview打开富文本并获取高度
author: 独孤流
date: 2024-08-17 01:04:00 +0800
categories: [webview]
tags: [webview]     # TAG names should always be lowercase
---

> 在暂时富文本弹窗内容时，为了方便展示适配，一般都直接使用wkwebview进行展示
> 同时好需要加载好内容后获取内容获取到真实的高度，如果高度超过一定的阈值，就要WebView高度固定变成可滚动

```
class TestWebVC: UIViewController, WKNavigationDelegate {
    var htmlWebViewHeight: CGFloat = 0
    var webView = WKWebView()
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.addSubview(webView)
        self.showText(content: "helloWorld")
    }
    // MARK: 加载文本
    func showText(content: String) {
        var html = """
          <html lang="zh-cn">
              <head>
                  <meta charset="utf-8"/>
                  <meta name="viewport" content="width=auto,initial-scale=1, maximum-scale=1, minimum-scale=1,user-scalable=no"/>
                  <style>
                      body {
                          font-size:100%;
                          margin:0;
                          padding-left:10;
                          padding-right:10;
                      }
                  </style>
              </head>
              <body>
                 #=#content#=#
              </body>
          </html>
        """
        // 将内容放到body里
        html = html.replacingOccurrences(of: "#=#content#=#", with: content)
        webView.loadHTMLString(html, baseURL: nil)
    }

    // MARK: WKNavigationDelegate
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        webView.evaluateJavaScript("document.body.scrollHeight") { result, error in
            guard let height = result as? CGFloat else { return }
            if self.htmlWebViewHeight == height {
                // 如果相同就不处理，防止无效浪费
                return
            }
            var showHeight: CGFloat = 0
            var isShowScollIndicator = false
            if height <= 50 {
                // 真实高度小于最小高度，直接使用最小高度
                showHeight = htmlWebViewMinHeight
            } else if height <= 250 {
                // 在最小最大之间，使用真实高度
                showHeight = height
            } else {
                // 超过了最大高度，使用最大高度，展示滚动条
                showHeight = 250
                isShowScollIndicator = true
            }
        
            // 计算出来的高度大于固定高度
            webView.scrollView.showsVerticalScrollIndicator = isShowScollIndicator
            webView.scrollView.isScrollEnabled = isShowScollIndicator
            // 更新WebView的高度
            webView.snp.updateConstraints { make in
                make.height.equalTo(showHeight)
            }
        }
    }
}
```