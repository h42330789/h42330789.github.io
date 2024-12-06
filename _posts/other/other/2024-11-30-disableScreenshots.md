---
title: iOS禁止截屏
author: 独孤流
date: 2024-11-30 00:04:00 +0800
categories: [other, 其他]
tags: [screenshot]     # TAG names should always be lowercase
---

参考：
- [iOS防截屏录屏｜担心App内容被截屏泄露吗？这个开源库就是你要的](https://juejin.cn/post/7066341701815631909)

1、不允许截屏：使用iOS系统自带的`UITextField`的`isSecureTextEntry`属性实现不允许截屏\
2、用户截屏后获取截屏通知：`UIApplication.userDidTakeScreenshotNotification`
```
class MyViewController: UIViewController {
        var isLImitMessage: Bool {
           didSet {
                changeScreenshot()
           }
        }
        
        override func viewDidLoad() {
            super.viewDidLoad()
            let btn = UIButton(type: cu)
            btn.setTitle("修改截屏状态", for: .normal)
            btn.setTitleColor(.black, for: .normal)
            btn.titleLabel?.numberOfLines = 0
            btn.frame = CGRectMake(10, 100, 100, 48)
            btn.backgroundColor = btnColor
            btn.addTarget(self, action: #selector(btnClick), for: .touchUpInside)
            self.view.addSubview(btn)
       }

        @objc func btnClick() {
          self.isLImitMessage = !self.isLImitMessage
        }


        var secureTextField: UITextField? = nil
        var secureView: UIView? = nil
        func changeScreenshot() {
            if (isLImitMessage) {
                if secureTextField != nil {
                    // 已经设置过了就不再继续设置
                    return
                }
                if secureTextField == nil {
                    let v = UITextField()
                    secureTextField = v
                    for subView in v.subviews {
                        let fullClassName = NSStringFromClass(type(of: subView))
                        if fullClassName.contains("TextLayoutCanvasView") {
                            secureView = subView
                            break
                        }
                    }
                }
                guard let secureView = secureView else {
                    return
                }
                let previousLayer = secureView.layer
                secureView.setValue(self.view.layer, forKey: "layer")
                secureTextField?.isSecureTextEntry = false
                secureTextField?.isSecureTextEntry = true
                secureView.setValue(previousLayer, forKey: "layer")
            } else {
                if secureTextField == nil {
                    // 已经设置过了就不再继续设置
                    return
                }
                if let previousLayer = secureView?.layer {
                    secureView?.setValue(self.view.layer, forKey: "layer")
                    secureTextField?.isSecureTextEntry = true
                    secureTextField?.isSecureTextEntry = false
                    secureView?.setValue(previousLayer, forKey: "layer")
                }
                if secureView != nil {
                    secureView = nil
                }
                if secureTextField != nil {
                    secureTextField = nil
                }
            }
        }
```

### Telegram的实现方式
```
    // 调用的地方
    setLayerDisableScreenshots(self.layer, self.isSecret)
    // 实现的地方
    void setLayerDisableScreenshots(CALayer * _Nonnull layer, bool disableScreenshots) {
        static UITextField *textField = nil;
        static UIView *secureView = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            textField = [[UITextField alloc] init];
            for (UIView *subview in textField.subviews) {
                if ([NSStringFromClass([subview class]) containsString:@"TextLayoutCanvasView"]) {
                    secureView = subview;
                    break;
                }
            }
        });
        if (secureView == nil) {
            return;
        }
        
        CALayer *previousLayer = secureView.layer;
        [secureView setValue:layer forKey:@"layer"];
        if (disableScreenshots) {
            textField.secureTextEntry = false;
            textField.secureTextEntry = true;
        } else {
            textField.secureTextEntry = true;
            textField.secureTextEntry = false;
        }
        [secureView setValue:previousLayer forKey:@"layer"];
        
        [layer setAssociatedObject:@(disableScreenshots) forKey:layerDisableScreenshotsKey associationPolicy:NSObjectAssociationPolicyRetain];
    }
}
```