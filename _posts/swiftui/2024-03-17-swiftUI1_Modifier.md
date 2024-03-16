---
title: SwiftUI1：系统Modifier和自定义Modifier
author: 独孤流
date: 2024-03-17 01:01:00 +0800
categories: [SwiftUI]
tags: [SwiftUI]     # TAG names should always be lowercase
---

参考：
- [HackingWithSwift](https://github.com/twostraws/HackingWithSwift)
- [[SwiftUI 知识碎片] Debris-13 环境 Modifier](https://juejin.cn/post/6844904058663206926)
- [[SwiftUI 知识碎片] Debris-15 自定义 modifier](https://juejin.cn/post/6844904063146917901)
- [StudyModifier.swift](https://github.com/h42330789/StudySwiftUI/blob/main/StudySwiftUI/StudySwiftUI/StudyModifier.swift)

### 一、 `SwiftUI`里的系统定义的`Modifier`
##### 1.1 子控件可优先级更高的Modifier，也就是子父控件和子控件同时设置，子控件优先级更高
```
VStack {
    Text("Gryffindor")
        .font(.largeTitle)
    Text("Hufflepuff")
    Text("Ravenclaw")
    Text("Slytherin")
}
.font(.title)
```
##### 1.2 子控件可优先级更高的Modifier，也就是子父控件和子控件同时设置，子控件优先级更高
```
VStack {
    Text("Gryffindor")
        .blur(radius: 0)
    Text("Hufflepuff")
    Text("Ravenclaw")
    Text("Slytherin")
}
.blur(radius: 5)

```
----
### 二、 `SwiftUI`里的自定义的`Modifier`
自定义需要自定义一个结构体，该结构体继承`ViewModifier`，在body里返回一个view对象

```
struct XXX: ViewModifier {
    func body(content: Content) -> some View {
        // ...
    }
}
```
##### Demo2.1: 只是修改原view的属性
```
struct Title: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.largeTitle)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .clipShape(RoundedRectangle(cornerRadius: 10))
    }
}

// 为了方便使用可以添加分类
extension View {
    func titleStyle() -> some View {
        self.modifier(Title())
    }
}

// 调用Demo
VStack {
        Text("Hello World")
            .modifier(Title()) // 直接调用modifier
        Text("Hello World22") // 使用扩展间接调用modifier
            .titleStyle()
    }

// 
```

##### Demo2.2: 返回一个新的view
```
struct Watermark: ViewModifier {
    var text: String

    func body(content: Content) -> some View {
        // 这里返回的是一个新的view
        ZStack(alignment: .bottomTrailing) {
            content
            Text(text)
                .font(.caption)
                .foregroundColor(.white)
                .padding(5)
                .background(Color.black)
        }
    }
}

extension View {
    func watermarked(with text: String) -> some View {
        self.modifier(Watermark(text: text))
    }
}

// 调用Demo
Color.blue
    .frame(width: 300, height: 200)
    .watermarked(with: "Hacking with Swift")
```
![image](/assets/img/swiftui/swiftui_modifier.png)