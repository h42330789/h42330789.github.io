---
title: 大量数据导致iOS因内存崩溃问题
author: 独孤流
date: 2024-10-28 00:04:00 +0800
categories: [other, 其他]
tags: [memery]     # TAG names should always be lowercase
---
参考：
- [https://github.com/kstenerud/KSCrash](https://github.com/kstenerud/KSCrash)

> #### 前言
> 目前线上有好几个用户反馈App用着用着就崩溃了，但是看搜集闪退日志的`Firebase`没有看到任何信息，后面一个同事集成了`https://github.com/kstenerud/KSCrash`后，让用户手动收集崩溃信息，发现是内存使用过度导致

1、参考文档使用`podfile`引入
2、在`appdelete`里引入
```
// 崩溃设置
let config = KSCrashConfiguration()

let report = CrashReportStoreConfiguration()
report.reportsPath = "\(NSSearchPathForDirectoriesInDomains(.cachesDirectory, .userDomainMask, true)[0])/aa/bbb"
report.appName = "MyApp"
config.reportStoreConfiguration = report
config.installPath = "(NSSearchPathForDirectoriesInDomains(.cachesDirectory, .userDomainMask, true)[0])/aa"
// 生成后的crash文件 /xxx/xxx/aa/bbb/MyApp-report-xxxx.json

let installation = CrashInstallationStandard.shared
try? installation.install(with: config)
```
3、使用数据,拿到data后上传
```
func readCrashData() -> Data? {
    let jsonArray = KSCrash.shared.reportStore?.reportIDs.map {
        var json = reportStore.report(for: $0.int64Value)
        return json
    }
    if let jsons =  jsonArray, jsons.count > 0 else {
        return try? KSJSONCodec.encode(jsons, options: .sorted)
    } else {
        return nil
    }
}
```
3.2 也可以直接拿到崩溃的路径，直接将路径里的文件全部打包上传

报错内容如下：
```
{
  "process" : {

  },
  "system" : {
    "binary_cpu_type" : 16777228,
    "boot_time" : null,
    "CFBundleName" : "xxxx",
    "app_uuid" : "xxxxxxx",
    "process_name" : "xxxx",
    "system_version" : "16.3.1",
    "device_app_hash" : "xxxx",
    "storage" : 0,
    "memory" : {
      "free" : 2812182528,
      "size" : 5918965760,
      "usable" : 5493374976
    },
    "application_stats" : {
      "background_time_since_last_crash" : 144.38300000000001,
      "active_time_since_launch" : 0.061150999999999997,
      "sessions_since_last_crash" : 75,
      "launches_since_last_crash" : 68,
      "active_time_since_last_crash" : 547.42100000000005,
      "application_active" : true,
      "sessions_since_launch" : 1,
      "application_in_foreground" : true,
      "background_time_since_launch" : 0
    },
    "CFBundleVersion" : "xxxx",
    "app_memory" : {
      "memory_level" : "critical",
      "memory_footprint" : 2980798272,
      "memory_remaining" : 240427200,
      "memory_limit" : 3221225472,
      "app_transition_state" : "active",
      "memory_pressure" : "normal",
      "timestamp" : xxxx
    },
    "cpu_type" : 16777228,
    "CFBundleShortVersionString" : "xxxx",
    "binary_cpu_subtype" : 0,
    "system_name" : "iOS",
    "app_start_time" : "xxxx",
    "CFBundleExecutable" : "xxxx",
    "process_id" : 1702,
    "cpu_subtype" : 2,
    "CFBundleIdentifier" : "xxxxx",
    "cpu_arch" : "arm64e",
    "model" : "xxx",
    "os_version" : "xxx",
    "jailbroken" : false,
    "time_zone" : "GMT+8",
    "CFBundleExecutablePath" : "xxxxx",
    "build_type" : "test",
    "machine" : "iPhone15,2",
    "kernel_version" : "Darwin Kernel Version 22.3.0: Wed Jan  4 21:25:01 PST 2023; root:xnu-8792.82.2~1\/RELEASE_ARM64_T8120",
    "parent_process_id" : 1
  },
  "crash" : {
    "error" : {
      "address" : 0,
      "memory_termination" : {
        "memory_level" : "critical",
        "memory_footprint" : 2980798272,
        "memory_remaining" : 240427200,
        "memory_limit" : 3221225472,
        "app_transition_state" : "active",
        "memory_pressure" : "normal",
        "timestamp" : xxx
      },
      "signal" : {
        "name" : "SIGKILL",
        "signal" : 9
      },
      "type" : "memory_termination"
    },
    "threads" : [

    ]
  },
  "report" : {
    "process_name" : "xxx",
    "id" : "xxxx",
    "timestamp" : xxxxx,
    "type" : "standard",
    "version" : "3.4.1"
  },
  "debug" : {

  },
  "user" : {

  }
}

```
核心内容：
```
 "app_memory" : {
      "memory_level" : "critical", // 严重
      "memory_footprint" : 2980798272, // 内存占用  2842 M
      "memory_remaining" : 240427200, // 剩余内存 229 M
      "memory_limit" : 3221225472, // 内存限制 3072M 
      "app_transition_state" : "active",
      "memory_pressure" : "normal",
      "timestamp" : xxxx
    },
"crash" : {
    "error" : {
      "address" : 0,
      "memory_termination" : {
        "memory_level" : "critical", // 严重
        "memory_footprint" : 2980798272, // 内存占用  2842 M
        "memory_remaining" : 240427200, // 剩余内存 229 M
        "memory_limit" : 3221225472, // 内存限制 3072M 
        "app_transition_state" : "active",
        "memory_pressure" : "normal",
        "timestamp" : xxxx
      },
      "signal" : {
        "name" : "SIGKILL",
        "signal" : 9
      },
      "type" : "memory_termination"
    },
    "threads" : [

    ]
  },
  ````