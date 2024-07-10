---
title: pod与spm的比较与区别(Amplify)
author: 独孤流
date: 2024-07-10 01:04:00 +0800
categories: [Pod_bazel_spm, Pod]
tags: [podsecp]     # TAG names should always be lowercase
---

参考：
- [amplify-2.33.6-package](https://github.com/aws-amplify/amplify-swift/blob/2.33.6/Package.swift)
- [amplify-1.31.0-package](https://github.com/aws-amplify/amplify-swift/blob/1.31.0/Package.swift)
- [Amplify-1.31.0-podspec-github](https://github.com/aws-amplify/amplify-swift/blob/1.31.0/Amplify.podspec)
- [Amplify-1.31.0-podspec-specs](https://github.com/CocoaPods/Specs/blob/master/Specs/9/c/5/Amplify/1.31.0/Amplify.podspec.json)


> ### 前言
> 最近在搞亚马逊人脸识别SDK接入，想把`spm`的方式改成`pod`的方式，改造起来十分痛苦都快抑郁了，特研究学习下`Amplify`的写法

### 一、Pod的配置
![pod_spm_amplify1.png](/assets/img/pod/pod_spm_amplify1.png)