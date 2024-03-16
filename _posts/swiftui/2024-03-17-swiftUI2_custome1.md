---
title: SwiftUI2：自定义View之将属性组合在一起便捷操作
author: 独孤流
date: 2024-03-17 01:01:00 +0800
categories: [SwiftUI]
tags: [SwiftUI]     # TAG names should always be lowercase
---

参考：
- [HackingWithSwift](https://github.com/twostraws/HackingWithSwift)
- [[SwiftUI 知识碎片] Debris-14 视图构成](https://juejin.cn/post/6844904061632774158)
- [StudyCutomView1.swift](https://github.com/h42330789/StudySwiftUI/blob/main/StudySwiftUI/StudySwiftUI/StudyCutomView1.swift)

由于相似的属性很多，代码显得很长，可以将一些相似的代码全封装起来，代码看起来会间接很多，想要将公共的代码组合在一起

方案一：自定义View
```
struct CapsuleText: View {
    var text: String

    var body: some View {
        Text(text)
            .font(.largeTitle)
            .padding()
            .foregroundColor(.white)
            .background(Color.blue)
            .clipShape(Capsule())
    }
}
struct CapsuleText2: View {
    var text: String

    var body: some View {
        Text(text)
            .font(.largeTitle)
            .padding()
            .background(Color.blue)
            .clipShape(Capsule())
    }
}
// 调用Demo
VStack(spacing: 10) {
        CapsuleText(text: "First")
        CapsuleText(text: "Second")
        CapsuleText2(text: "Third")
                .foregroundColor(.white)
            CapsuleText2(text: "Fouth")
                .foregroundColor(.yellow)
    }
```
方案二：使用`Modifier`
```
// MARK: - 使用Modifier进行组合
struct MyCapsule: ViewModifier {
    var foregroundColor: Color
    func body(content: Content) -> some View {
        content
            .font(.largeTitle)
            .padding()
            .foregroundColor(foregroundColor)
            .background(Color.blue)
            .clipShape(Capsule())
    }
}

extension View {
    func myCapsuleStyle(_ foregroundColor: Color = .white) -> some View {
        self.modifier(MyCapsule(foregroundColor: foregroundColor))
    }
}
// 调用Demo
VStack(spacing: 10) {
        Text("First1").myCapsuleStyle()
        Text("Second1").myCapsuleStyle()
        Text("Third1").myCapsuleStyle(.red)
        Text("Fouth1").myCapsuleStyle(.green)
    }
```
![image](/assets/img/swiftui/swiftui_custom1.png)