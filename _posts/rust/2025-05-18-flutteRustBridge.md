---
title: Rust项目代码与flutter桥接
author: 独孤流
date: 2025-05-18 13:14:00 +0800
categories: [rust]
tags: [iOS]     # TAG names should always be lowercase
---

rust由于是跨平台的，可以提供给iOS、安卓等其他平台封装的库，同时如果UI层是flutter，还可以直接使用`flutter_rust_bridge`将rust代码自动生成文件桥接

GitHub：https://github.com/fzyzcjy/flutter_rust_bridge

示例项目：https://github.com/fzyzcjy/flutter_rust_bridge/tree/main/frb_example

```
cargo install flutter_rust_bridge_codegen
```

dart-dartbridge - ffi - rust.framework
swift - ffi - rust.framework

ffi: Foreign Function Interface （跨语言函数接口）

vscode
cursor
flutter
rust
flutter_rust_bridge_codegen --> rust-dart
prost --> protobuf