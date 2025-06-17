---
title: FlutterRustBridge异步调用与监听回调
author: 独孤流
date: 2025-05-20 10:14:00 +0800
categories: [rust]
tags: [iOS]     # TAG names should always be lowercase
---

#### 一、准备工作
flutter调用通过flutter_rust_bridge调用
1、在rust的`Cargo.toml`配置
```
[build-dependencies]
flutter_rust_bridge_codegen = "2.9.0"
```
2、`flutter的pubspec.ymal`配置
```
dependencies
    flutter_rust_bridge: 2.9.0
```
3、配置cli命令
```
cargo install flutter_rust_bridge_codegen --version 2.9.0
```

#### 二、当其他都配置好后
1、运行命令会自动生成对应代码
```
flutter_rust_bridge_codegen  generate

# 会自动在flutter项目里生成
# frb_generated.dart
# 会自动在rust项目里生成
# frb_generated.rs
```

rust项目里写的异步方法：
```
pub async fn login_by_password(
    account: String
    password: String,
) -> Result<LoginResponse> {
    Ok(LoginResponse{xxxxx})
}
```
frb_generated.dart 会在每次调用都生成唯一的端口号，端口号对应一个回调，rust处理完后，会执行这个回调，跟网络请求类似


一般方法调用，使用`ReceivePort`的方式
```
          Dart (Flutter)
             ▲       │
             │       ▼
         port       FFI
             │       ▲
             ▼       │
             Rust (native)

```
| 环节          | 行为     | 技术机制                              |
| ----------- | ------ | --------------------------------- |
| Dart → Rust | 发起异步调用 | FFI + Dart Future                 |
| Rust → Dart | 异步回调结果 | Dart Port + Dart\_PostCObject\_DL |
| Dart        | 处理返回   | Port 监听器 resolve Future           |


长连接的监听使用`StreamSink`

| 项目   | 普通 FFI 调用（一次性）        | `StreamSink`（持续通信）                    |
| ---- | --------------------- | ------------------------------------- |
| 通信通道 | 使用 Dart 的 Port（会自动释放） | 使用 Dart 的 Port（**不会主动关闭**）            |
| 生命周期 | 调用完成后关闭 Port          | 需要手动 `sink.close()` 才关闭               |
| 数据模型 | 一次性 Future / 单值返回     | 多次 `sink.add(...)`，对应 Dart 的 `Stream` |
| 场景   | 调用函数、取结果              | 长连接、事件流、状态更新等                         |
