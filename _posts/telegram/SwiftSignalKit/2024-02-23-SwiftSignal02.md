---
title: SwiftSignalKit研究二：其他常用方法
author: 独孤流
date: 2024-02-23 01:04:00 +0800
categories: [Telegram, SwiftSignalKit]
tags: [Telegram, SwiftSignalKit]     # TAG names should always be lowercase
---

- [Operator - 操作符](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core/operator.html)
- [如何选择操作符？](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree.html)
  
### 操作符（Operators）
```
precedencegroup PipeRight {
    associativity: left
    higherThan: DefaultPrecedence
}

infix operator |> : PipeRight

public func |> <T, U>(value: T, function: ((T) -> U)) -> U {
    return function(value)
}
```

### ValuePipe
### Timer

`Catch`: 
`restart`:
`recurse`:
`retry`:
`restartIfError`: