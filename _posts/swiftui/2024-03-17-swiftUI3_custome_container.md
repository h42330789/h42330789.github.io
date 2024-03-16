---
title: SwiftUI3：自定义容器
author: 独孤流
date: 2024-03-17 01:01:00 +0800
categories: [SwiftUI]
tags: [SwiftUI]     # TAG names should always be lowercase
---

参考：
- [HackingWithSwift](https://github.com/twostraws/HackingWithSwift)
- [[SwiftUI 知识碎片] Debris-16 自定义容器](https://juejin.cn/post/6844904063738314766)
- [StudyCutomContainer.swift](https://github.com/h42330789/StudySwiftUI/blob/main/StudySwiftUI/StudySwiftUI/StudyCutomContainer.swift)

使用泛型做一些很强大的封装
```
struct GridStack<Content: View>: View {
    let rows: Int
    let columns: Int
    let content: (Int, Int) -> Content

    var body: some View {
        // more to come
    }
}
```
>第一行 —— struct GridStack<Content: View>: View —— 用一个 Swift 的高级特别叫 泛型，在这里它的意思是 “你可以提供任意类型的内容，但它必须遵循 View 协议。” 在冒号之后，我们又加了一个 View ，声明 GridStack 本身也遵循View 协议。特别注意一下 let content 这行 —— 它定义了一个闭包 —— 必须接收两个整数，并且返回某种我们可以显示的内容。我们需要完成 body 属性 —— 通过组合多个 vertical 和 horizontal 的 stack 来按要求创建许多单元。我们不需要说明每个单元里有什么，因为我们可以通过合适的行号和列号来调用 content 闭包


完整Demo
```
struct GridStack<Content: View>: View {
    let rows: Int
    let columns: Int
    let content: (Int, Int) -> Content

    var body: some View {
        VStack {
            ForEach(0 ..< rows, id: \.self) { row in
                HStack {
                    ForEach(0 ..< self.columns, id: \.self) { column in
                        self.content(row, column)
                    }
                }
            }
        }
    }

}
// 调用Demo
GridStack(rows: 4, columns: 4) { row, col in
        Text("R\(row) C\(col)")
    }
```
### 使用ViewBuilder创建, ViewBuilder会自动将多个视图组创建一个stack
```
struct GridStack2<Content: View>: View {
    let rows: Int
    let columns: Int
    let content: (Int, Int) -> Content
    
    init(rows: Int, columns: Int, @ViewBuilder content: @escaping (Int, Int) -> Content) {
        self.rows = rows
        self.columns = columns
        self.content = content
    }


    var body: some View {
        VStack {
            ForEach(0 ..< rows, id: \.self) { row in
                HStack {
                    ForEach(0 ..< self.columns, id: \.self) { column in
                        self.content(row, column)
                    }
                }
            }
        }
    }

}
// 调用Demo
GridStack2(rows: 4, columns: 4) { row, col in
            Image(systemName: "\(row * 4 + col).circle")
            Text("R\(row) C\(col)")
        }
```
![image](/assets/img/swiftui/swiftui_container1.png)