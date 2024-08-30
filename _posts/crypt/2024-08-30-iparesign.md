---
title: 防止Ipa包被重签名
author: 独孤流
date: 2024-08-30 01:12:00 +0800
categories: [加解密]
tags: [重签名]      # TAG names should always be lowercase
---
参考：
- [iOS证书签名机制&重签名&防止重签名](https://juejin.cn/post/6844904013159202824)
- [iOS开发安全 - 防重签名、越狱、调试检测](https://blog.csdn.net/guoxulieying/article/details/131787414)

问题一：APP被修改名称、版本号重新签名

问题二：业务域名被疯狂攻击

问题三、socket服务被疯狂攻击