---
title: SwiftUI4：Binding
author: 独孤流
date: 2024-03-17 01:01:00 +0800
categories: [SwiftUI]
tags: [SwiftUI]     # TAG names should always be lowercase
---

参考：
- [HackingWithSwift](https://github.com/twostraws/HackingWithSwift)
- [[SwiftUI 知识碎片] 结构体和类，ForEach，绑定](https://juejin.cn/post/6844904070952517640)
- [StudyBinding.swift](https://github.com/h42330789/StudySwiftUI/blob/main/StudySwiftUI/StudySwiftUI/StudyBinding.swift)

使用Binding属性可以对属性的变化进行监听及相关业务逻辑处理
```
// 不使用binding
struct ContentView1: View {
    @State var selection = 0

    var body: some View {
        return VStack {
            Picker("Select a number", selection: $selection) {
                ForEach(0 ..< 3) {
                    Text("Item \($0)")
                }
            }.pickerStyle(SegmentedPickerStyle())
            Text("selected: \(selection)")
        }
    }
}
// 使用binding拦截get/set
struct ContentView2: View {
    @State var selection = 0

    var body: some View {
        let binding = Binding(
            get: { self.selection },
            set: { self.selection = $0 }
        )

        return VStack {
            Picker("Select a number", selection: binding) {
                ForEach(0 ..< 3) {
                    Text("Item \($0)")
                }
            }.pickerStyle(SegmentedPickerStyle())
            Text("selected: \(selection)")
        }
    }
}
// 使用binding做复杂的组合业务逻辑
struct ContentView3: View {
    @State var agreedToTerms = false
    @State var agreedToPrivacyPolicy = false
    @State var agreedToEmails = false

    var body: some View {
        let agreedToAll = Binding<Bool>(
            get: {
                self.agreedToTerms && self.agreedToPrivacyPolicy && self.agreedToEmails
            },
            set: {
                self.agreedToTerms = $0
                self.agreedToPrivacyPolicy = $0
                self.agreedToEmails = $0
            }
        )

        return VStack {
            Toggle(isOn: $agreedToTerms) {
                Text("Agree to terms")
            }

            Toggle(isOn: $agreedToPrivacyPolicy) {
                Text("Agree to privacy policy")
            }

            Toggle(isOn: $agreedToEmails) {
                Text("Agree to receive shipping emails")
            }

            Toggle(isOn: agreedToAll) {
                Text("Agree to all")
            }
            Text("terms: \(agreedToTerms ? "true" : "false")")
            Text("policy: \(agreedToPrivacyPolicy ? "true" : "false")")
            Text("emails: \(agreedToEmails ? "true" : "false")")
            Text("all: \(agreedToAll.wrappedValue ? "true" : "false")")
        }
    }
}

// 调用demo
#Preview {
    VStack(spacing: 10) {
        ContentView1()
        ContentView2()
        ContentView3()
    }
}

```
![image](/assets/img/swiftui/swiftui_binding.png)