---
title: iOS延迟执行时间的判断
author: 独孤流
date: 2024-06-22 01:04:00 +0800
categories: [Tool]
tags: [Tool,Swift]     # TAG names should always be lowercase
---

> ### 前言
> 在开发过程中，经常有些东西需要延迟执行，但是容易把延迟执行的写法理解有误，特记录

```
extension Date {
    static func systemMilliseconds_int64() -> Int64 {
        return Int64(Date().timeIntervalSince1970*1000)
    }
}

// 延迟时间与设置的延迟时间会有轻微的差异，大体是没问题的

let now0 = Date.systemMilliseconds_int64()
print("==**>>--AA-->>: now0:\(now0)")
// ==**>>--AA-->>: now0:1719038441921

// .now()+数字，数字代表秒数，可以写整数，也可以写小数，表示延迟xxx秒后执行
DispatchQueue.global().asyncAfter(deadline: .now() + 0.5 , execute: {
    let now1 = Date.systemMilliseconds_int64()
    print("==**>>--BB1-->>: now0:\(now0) now1:\(now1) dis:\(now1-now0)")
    // ==**>>--BB1-->>: now0:1719038441921 now1:1719038442422 dis:501
})
DispatchQueue.global().asyncAfter(deadline: .now() + 1, execute: {
    let now1 = Date.systemMilliseconds_int64()
    print("==**>>--BB2-->>: now0:\(now0) now1:\(now1) dis:\(now1-now0)")
    // ==**>>--BB2-->>: now0:1719038441921 now1:1719038442971 dis:1050
})

// .now() + .seconds(xxx), xxx必须是整数，.seconds(xxx)就是延迟xxx秒执行
DispatchQueue.global().asyncAfter(deadline: .now() + .seconds(5) , execute: {
    let now2 = Date.systemMilliseconds_int64()
    print("==**>>--CC-->>: now0:\(now0) now2:\(now2) dis:\(now2-now0)")
    // ==**>>--CC-->>: now0:1719038441921 now2:1719038447171 dis:5250
})

// .now() + .milliseconds(xxx), xxx必须是整数，.milliseconds(xxx)就是延迟xxx毫秒执行
DispatchQueue.global().asyncAfter(deadline: .now() + .milliseconds(300) , execute: {
    let now3 = Date.systemMilliseconds_int64()
    print("==**>>--DD-->>: now0:\(now0) now3:\(now3) dis:\(now3-now0)")
    // ==**>>--DD-->>: now0:1719038441921 now3:1719038442224 dis:303
})
// 还有微妙、纳秒，很少使用，就不列出来了
```