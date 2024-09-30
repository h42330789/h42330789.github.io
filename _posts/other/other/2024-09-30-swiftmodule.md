---
title: Could not find module 'xxx' for target 'x86_64-apple-ios-simulator'
author: 独孤流
date: 2024-09-30 01:04:00 +0800
categories: [other, 其他]
tags: [libarclite]     # TAG names should always be lowercase
---

> ` Could not find module 'xxx' for target 'x86_64-apple-ios-simulator'; found: arm64-apple-ios-simulator, at: /Users/xxx/Library/Developer/Xcode/DerivedData/ExamplePod-gnqhkdbrzigmizayegbigopeyeut/Index.noindex/Build/Products/Debug-iphonesimulator/xxx/xxx.framework/Modules/xxx.swiftmodule`

https://github.com/facebook/react-native/issues/37627

修改`podfile`文件
```
installer.pods_project.build_configurations.each do |config|
      config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"
end
```
完整demo如下：
```
platform :ios, '9.0'

target 'Example' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for Example
  pod 'LookinServer', :configurations => ['Debug']

  target 'ExampleTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'ExampleUITests' do
    # Pods for testing
  end
  
  post_install do |installer|
    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"
      end
    end
  end

end
```