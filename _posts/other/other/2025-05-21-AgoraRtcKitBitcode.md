---
title: 最新xcode上传ipa到AppStore提示
author: 独孤流
date: 2025-05-14 10:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

参考：
- [App Store Connect Upload Failure Invalid Executable. The executable contains bitcode.](https://github.com/AgoraIO-Extensions/Agora-Flutter-RTM-SDK/issues/176)

- [Fixing Bitcode Issues in Xcode 16: How to Resolve Invalid Executable Errors When Uploading iOS Builds](https://medium.com/@abdulahad2024/fixing-bitcode-issues-in-xcode-16-how-to-resolve-invalid-executable-errors-when-uploading-ios-da07a5a39c7c)


> ### 前言
> 最近AppStore提示2025年4月24号开始，新上传的APP必须要在Xcode16及以上版本上传
```
Uploaded with warnings
    App Store Connect Warning
    SDK version issue. This app was built with the iOS 17.4 SDK. Starting April 24, 2025, all iOS and iPadOS apps must be built with the iOS 18 SDK or later, include in Xcode 16 or later, in order to  be uploaded to App Store Connect or submitted for distribution.
```

![image](/assets/img/other/appstore_update.png)


根据提示升级了`MacOS`和`Xcode16.3`，升级后上传包，发现有又出了另一个问题
```
Upload failed
    Validation failed

    Invalid Executable. The executable 'xxxx.app/Frameworks/AgoraRtcCryptoLoader.framework/AgoraRtcCrytoLoader' contains bitcode. (ID: xxxxxxxxxxx)

    Validation failed

    Invalid Executable. The executable 'xxxx/app/Frameworks/AgoraRtcKit.framework/AgoraRtcKit' contains bitcode. (Id: xxxxxxxx)
```
![image](/assets/img/other/appstore_bitcode.png)

查了下是因为使用的`pod 'AgoraRtcEngine_iOS_Crypto', '~> 3.1.2'`

版本比较旧，还包含`bitcode`，如果要升级过高版本，SDK接口变化比较大，而且跟之前的旧版本或其他端没法通话和视频，为了最少变化且保持跟其他旧版本能正常使用，采用了动态删除bitcode的方法，具体逻辑如下：

`Podfile`
```
platform :ios, '12.0'
  source 'https://github.com/CocoaPods/Specs.git'

  target 'xxxx' do 
      inhibit_all_warnings!
      use_frameworks!

      pod 'Alamofire', '~> 4.9.1'
      #声网
      pod 'AgoraRtcEngine_iOS_Crypto', '~> 3.1.2'
  end
  


  post_install do |installer|
    
    bitcode_strip_path = `xcrun --find bitcode_strip`.chop!
      def strip_bitcode_from_framework(bitcode_strip_path, framework_relative_path)
        framework_path = File.join(Dir.pwd, framework_relative_path)
        command = "#{bitcode_strip_path} #{framework_path} -r -o #{framework_path}"
        puts "Stripping bitcode: #{command}"
        system(command)
      end

    framework_paths = [
"Pods/AgoraRtcEngine_iOS_Crypto/Agora_Native_SDK_for_iOS_FULL/libs/AgoraRtcCryptoLoader.framework/AgoraRtcCryptoLoader",
"Pods/AgoraRtcEngine_iOS_Crypto/Agora_Native_SDK_for_iOS_FULL/libs/AgoraRtcKit.framework/AgoraRtcKit",
    ]

    framework_paths.each do |framework_relative_path|
      strip_bitcode_from_framework(bitcode_strip_path, framework_relative_path)
    end
  end

```

