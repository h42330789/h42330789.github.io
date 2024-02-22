---
title: Telegram研究四：错误整理
author: 独孤流
date: 2024-02-17 01:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram]     # TAG names should always be lowercase
---

项目： [https://github.com/TelegramMessenger/Telegram-iOS](https://github.com/TelegramMessenger/Telegram-iOS)\
Telegram-tag：`release-9.6.5`\
电脑：芯片：`M1`，系统：`MacOS Sonoma 14.0`，XCode：`15.0.1 `\
本demo项目：[https://github.com/h42330789/StudyIM](https://github.com/h42330789/StudyIM)\
本demo分支：`origin/feature/ItemListController`
----

### 问题一："The file “swiftc” doesn’t exist."
```
error: Could not parse Swift versions from: Swift/ErrorType.swift:200: Fatal error: Error raised at top level: Error Domain=NSCocoaErrorDomain Code=4 "The file “swiftc” doesn’t exist." UserInfo={NSFilePath=swiftc}

error: Could not parse Swift versions from: Swift/ErrorType.swift:200: Fatal error: Error raised at top level: Error Domain=NSCocoaErrorDomain Code=4 "The file “swiftc” doesn’t exist." UserInfo={NSFilePath=swiftc}

error: Could not parse Swift versions from: Swift/ErrorType.swift:200: Fatal error: Error raised at top level: Error Domain=NSCocoaErrorDomain Code=4 "The file “swiftc” doesn’t exist." UserInfo={NSFilePath=swiftc}
```

解决方案
使用`release-10.1`以上的tag或直接编译`master`分支

----

### 问题二 找不到预定义的宏

在引入`lottie`时,将`Bazel`改为`Pod`引入，在Bazel的文件夹里定义了预定义的宏，但是Pod里没有写，导致对应的宏不存在,报一系列错误
```
Use of undeclared identifier 'pixman_region_selfcheck'
```

![image](/assets/img/telegram/telegram_error_define1.png)

其他错误
```
ld: Undefined symbols:
  Vcomp_func_solid_SourceOver_neon(unsigned int*, int, unsigned int, unsigned int), referenced from:
      vInitDrawhelperFunctions() in RLottieBinding[23](vdrawhelper.o)
  memfill32(unsigned int*, unsigned int, int), referenced from:
      comp_func_solid_Source(unsigned int*, int, unsigned int, unsigned int) in RLottieBinding[19](vcompositionfunctions.o)
      fetch_linear_gradient(unsigned int*, Operator const*, VSpanData const*, int, int, int) in RLottieBinding[23](vdrawhelper.o)
      fetch_radial_gradient(unsigned int*, Operator const*, VSpanData const*, int, int, int) in RLottieBinding[23](vdrawhelper.o)
      blendColorARGB(unsigned long, VRle::Span const*, void*) in RLottieBinding[23](vdrawhelper.o)
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

![image](/assets/img/telegram/telegram_error_define2.png)
![image](/assets/img/telegram/telegram_error_define3.png)

`BUILD`的预编译配置如下：
```
objc_library(
    name = "RLottieBinding",
    ...
    copts = [
        "-Dpixman_region_selfcheck(x)=1",
        "-DLOTTIE_DISABLE_ARM_NEON=1",
        "-DLOTTIE_THREAD_SAFE=1",
        "-DLOTTIE_IMAGE_MODULE_DISABLED=1",
        "-I{}".format(package_name()),
        "-I{}/rlottie/inc".format(package_name()),
        "-I{}/rlottie/src/vector".format(package_name()),
        "-I{}/rlottie/src/vector/pixman".format(package_name()),
        "-I{}/rlottie/src/vector/freetype".format(package_name()),
    ],
    ...
)
```
改写成`xxx.podspec`的宏配置配置如下：
```
s.pod_target_xcconfig = {
    'GCC_PREPROCESSOR_DEFINITIONS' => 'pixman_region_selfcheck(x)=1 LOTTIE_DISABLE_ARM_NEON=1 LOTTIE_THREAD_SAFE=1 LOTTIE_IMAGE_MODULE_DISABLED=1'
}
```

完整的`podspec`如下：
```
Pod::Spec.new do |s|

    s.name         = "RLottieBinding"
    s.version      = "1.0.0"
    s.authors      = { 'Huy Nguyen' => 'hi@huynguyen.dev', 'Garrett Moon' => 'garrett@excitedpixel.com', 'Scott Goodson' => 'scottgoodson@gmail.com', 'Michael Schneider' => 'mischneider1@gmail.com', 'Adlai Holler' => 'adlai@icloud.com' }
    s.summary      = 'Smooth asynchronous user interfaces for iOS apps.'
    s.source        = { :git => "https://github.com/TextureGroup/Texture.git", :tag => s.version.to_s }
    
    s.license = 'MIT'
    s.homepage = 'http://www.example.com'
    s.requires_arc  = true
    s.ios.deployment_target = "12.0"

    s.public_header_files = 'PublicHeaders/**/*.h'
    # s.source_files =  ['rlottie/src/**/*.cpp', 'rlottie/src/**/*.h', 'rlottie/inc/**/*.h', 'PublicHeaders/**/*.h', 'LottieInstance.mm', 'config.h','boost/**/*.{h,hpp}']
    s.source_files =  ['rlottie/src/**/*.cpp', 'rlottie/src/**/*.h', 'rlottie/inc/**/*.h', 'PublicHeaders/**/*.h', 'LottieInstance.mm', 'config.h']
    s.exclude_files = ["rlottie/src/vector/vdrawhelper_neon.cpp", "rlottie/src/vector/stb/**/*", "rlottie/src/lottie/rapidjson/msinttypes/**/*"]

    # s.pod_target_xcconfig = {
    #     'GCC_PREPROCESSOR_DEFINITIONS' => '$(inherited) pixman_region_selfcheck(x)=1 LOTTIE_DISABLE_ARM_NEON=1 LOTTIE_THREAD_SAFE=1 LOTTIE_IMAGE_MODULE_DISABLED=1'
    # }
    s.pod_target_xcconfig = {
        'GCC_PREPROCESSOR_DEFINITIONS' => 'pixman_region_selfcheck(x)=1 LOTTIE_DISABLE_ARM_NEON=1 LOTTIE_THREAD_SAFE=1 LOTTIE_IMAGE_MODULE_DISABLED=1'
    }
    s.libraries = 'c++'
end

```
----

### 问题三、由于同时存在x86芯片的电脑和M系列芯片电脑，所以需要对.a进行合并
```
#!/bin/bash
# 合并架构
lipo -create ./arm/libsharpyuv.a ./x86/libsharpyuv.a -output ./full/libsharpyuv.a
# 查看架构  
lipo -info ./full/libsharpyuv.a
```