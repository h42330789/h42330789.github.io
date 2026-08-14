---
title: 已有项目接入Flutter App的软连接module
author: 独孤流
date: 2025-06-30 12:12:00 +0800
categories: [flutter]
tags: [flutter]     # TAG names should always be lowercase
---

加上目录
```
my_project
├── flutter_app
│   ├── README.md
│   ├── README_emotion_service.md
│   ├── analysis_options.yaml
│   ├── android
│   ├── assets
│   ├── build
│   ├── build.yaml
│   ├── flutter_build_runner.sh
│   ├── flutter_rust_bridge.yaml
│   ├── ios
│   ├── lib
│   ├── pubspec.lock
│   ├── pubspec.yaml
│   ├── rust_builder
│   ├── src
│   └── src
└── rust_lib
    ├── Cargo.lock
    ├── Cargo.toml
    ├── README.md
    ├── build.rs
    ├── cursor_rules
    ├── src
    └── tests
```

```
# 进入my_project
cd my_project
# 创建module项目
flutter create --template module flutter_module
# 进入创建的module项目
cd flutter_module
# 删除默认文件，创建软连接指向flutter_app
rm -rf lib
ln -s ../flutter_app/lib lib
rm pubspec.yaml
ln -s ../flutter_app/pubspec.yaml pubspec.yaml
rm -rf assets
ln -s ../flutter_app/assets assets
rm analysis_options.yaml
ln -s ../flutter_app/analysis_options.yaml analysis_options.yaml
# 由于flutter_app里引入了rust库，所以这里需要给生成的rust项目也进行软连接管理
rm -rf rust_builder
ln -s ../openchat_client/rust_builder rust_builder
# 执行编译
flutter clean
flutter pub get
```

配置flutter_rust_bridge的生成项目
xxx/rust_builder/cargokit/build_pod.sh
xxx/rust_builder/cargokit/run_build_tool.sh
都在最顶部配置flutter位置
```
#!/usr/bin/env bash

set -e
if [ -z "$FLUTTER_ROOT" ]; then
  # which flutter的结果 /Users/xxx/flutter/bin/flutter
  FLUTTER_ROOT="/Users/xxx/flutter"  # fallback 路径
fi
export PATH="$FLUTTER_ROOT/bin:$FLUTTER_ROOT/bin/cache/dart-sdk/bin:$PATH"
.
```

操作之后的结果
```
my_project
├── flutter_module
│   ├── README.md
│   ├── analysis_options.yaml -> ../flutter_app/analysis_options.yaml
│   ├── assets -> ../flutter_app/assets
│   ├── flutter_module.iml
│   ├── flutter_module_android.iml
│   ├── lib -> ../flutter_app/lib
│   ├── pubspec.lock
│   ├── pubspec.yaml -> ../flutter_app/pubspec.yaml
│   ├── rust_builder -> ../flutter_app/rust_builder
│   └── test
├── flutter_app
│   ├── README.md
│   ├── README_emotion_service.md
│   ├── analysis_options.yaml
│   ├── android
│   ├── assets
│   ├── build
│   ├── build.yaml
│   ├── flutter_build_runner.sh
│   ├── flutter_rust_bridge.yaml
│   ├── ios
│   ├── lib
│   ├── pubspec.lock
│   ├── pubspec.yaml
│   ├── rust_builder
│   ├── src
│   └── src
└── rust_lib
    ├── Cargo.lock
    ├── Cargo.toml
    ├── README.md
    ├── build.rs
    ├── cursor_rules
    ├── src
    └── tests
```

在swift项目的`Podfile`里引入`flutter_module`
```
# Uncomment the next line to define a global platform for your project
platform :ios, '12.0'
source 'https://github.com/CocoaPods/Specs.git'

# flutter配置1找到当前Podfile相对于module的相对路径
flutter_application_path = '../flutter_module'
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

abstract_target 'abstract_pod' do  #这里的abstract_pod在实际targets中不存在，是虚拟
   inhibit_all_warnings!
   use_frameworks!
   
   pod 'Alamofire', '~> 4.9.1'
   pod 'SnapKit', '~> 5.0.1'
   pod 'Kingfisher', '~> 5.14.1'

  target 'HelloWorld' do
    # flutter配置2（必须卸载target里，如果写在abstract_target里会报错 [!] Invalid `Podfile` file: [!] Script phases cannot be added to abstract targets..）
    # install_all_flutter_pods(flutter_application_path)
  end

end

post_install do |installer|
    # flutter配置3
    flutter_post_install(installer) if defined?(flutter_post_install)
    installer.pods_project.targets.each do |target|
      if target.name == 'BoringSSL-GRPC'
        # 解决BoringSSL-GRPC报错问题
        target.source_build_phase.files.each do |file|
          if file.settings && file.settings['COMPILER_FLAGS']
            flags = file.settings['COMPILER_FLAGS'].split
            flags.reject! { |flag| flag == '-GCC_WARN_INHIBIT_ALL_WARNINGS' }
            file.settings['COMPILER_FLAGS'] = flags.join(' ')
          end
        end
      end
      target.build_configurations.each do |config|
        # 解决M系列芯片报错问题
        config.build_settings['ONLY_ACTIVE_ARCH'] = 'NO'
        config.build_settings['LD_NO_PIE'] = 'NO'
        config.build_settings['CODE_SIGNING_ALLOWED'] = 'NO'
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '12.0'
      end
    end
end
```

----
接入失败，直接在flutter_module里的pubspec.yaml里引入
```
name: flutter_module
description: "A new Flutter module project."

version: 1.0.0+1

environment:
  sdk: ^3.7.2

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^1.0.8

  openchat_client:
    path: ../flutter_app

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0


flutter:
  
  module:
    androidX: true
    androidPackage: com.example.flutter_module
    iosBundleIdentifier: com.example.flutterModule

```
执行脚本
```
flutter pub get
flutter build ios-framework
```

SEVERE:   --- stderr
SEVERE:   Error: Custom { kind: Other, error: Custom { kind: NotFound, error: "Could not find `protoc`. If `protoc` is installed, try setting the `PROTOC` environment variable to the path of the `protoc` binary. To install it on macOS, run `brew install protobuf`. It is also available at https://github.com/protocolbuffers/protobuf/releases  For more information: https://docs.rs/prost-build/#sourcing-protoc" } }

build.rs
fn main() {
    // 默认路径找不到时自己设定
    if env::var("PROTOC").is_err() {
        env::set_var("PROTOC", "/opt/homebrew/bin/protoc");
    }
    
    // ... 其他编译逻辑
}