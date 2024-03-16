---
title: SwiftUI自定义及各个属性
author: 独孤流
date: 2024-03-09 01:04:00 +0800
categories: [SwiftUI]
tags: [SwiftUI]     # TAG names should always be lowercase
---

参考：
- [swiftui-state-property-binding-stateobject-observedobject-environmentobject-學習筆記](https://medium.com/%E5%BD%BC%E5%BE%97%E6%BD%98%E7%9A%84-swift-ios-app-%E9%96%8B%E7%99%BC%E6%95%99%E5%AE%A4/swiftui-state-property-binding-stateobject-observedobject-environmentobject-%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98-b4366f87f4f4)
- [SwiftUI_learning01](https://github.com/jasonyen1009/SwiftUI_learning01)
- [探讨 SwiftUI 中的关键属性包装器：@State、@Binding、@StateObject、@ObservedObject、@EnvironmentObject 和 @Environment](https://fatbobman.com/zh/posts/exploring-key-property-wrappers-in-swiftui/)
- [SwiftUI教程系列文章汇总](https://juejin.cn/post/7110918270743478279)
- [SwiftUI教程（七）属性包装器：State、Binding、ObservableObject、EnvironmentObject](https://juejin.cn/post/7112984613102092325)
- [iOS Combine：核心概念](https://zhiying.space/posts/ios-combine-core-concepts/)
- [[SwiftUI 知识碎片] 为什么 @State 只能在结构体中工作](https://juejin.cn/post/6844904086823763982)

`var xxx = xxx`: 普通变量，只能读\
`@State var xxx = xxx`: 当前页面的简直值变量，需要有初始值，可使用，可修改\
`@Binding var xxx`: 当前页面的引用变量，不需要有初始化值，有外面传入，可使用，可修改，修改后外侧也同步修改\
`class xxx: ObservableObject`+ `@Published var xx = xx`: 用于传递的复杂对象，可以在页面之间传递\
`@StateObject var xxx = xxx`: 复杂的对象，需要有初始值，可使用，可修改\
`@ObservedObject var xxx`: 复杂的对象，不需要初始值，需要由外侧传入，可使用，可修改\
`.environmentObject(xxx)`+ `@EnvironmentObject var xxx: YYY`: 用于传递的复杂对象，可以在页面之间传递,可使用修改，不用显示传递\
```
import SwiftUI

struct MyContentView: View {
    
    // 普通的只读变量
    var isFillA = false
    // 由于SwiftUI使用struct定义，在struct中要改变property必须在var前增加@State
    @State var isFillB = false
    // 由于SwiftUI使用struct定义，在struct中要改变property必须在var前增加@State, 为了更安全，可以在state前添加private
    @State private var isFill = false
    var body: some View {
        
        VStack {
            Image(systemName: isFill ? "heart.fill" : "heart")
                .resizable()
                .scaledToFit()
            
            Button {
                isFill.toggle() // 与 isFill = !isFill 一致
                
            } label: {
                Text("press")
            }
            
        }
    }
}
// MARK: - @State+@Binding
struct PlayerButton: View {
    @State var play = true // @State相当于值传递，修改了值只会在本类中有影响，需要设置初始值
    var body: some View {
        VStack {
            Button {
                // 由于play是@Binding修饰，里面修改，只会影响到本类里
                play.toggle()
            }label: {
                Image(systemName: play ? "play.circle" : "pause.circle")
            }
            .font(.title)
        }
    }
}
struct PlayerButton2: View {
    @Binding var play: Bool // @Binding相当于引用传递，修改了值所有都会一起修改，不需要初始值，
    var body: some View {
        VStack {
            Button {
                // 由于play是@Binding修饰，里面修改，也会触发传入的那个值修改，相当于引用传递
                play.toggle()
            }label: {
                Image(systemName: play ? "play.circle" : "pause.circle")
            }
            .font(.title)
        }
    }
}

struct BindingView: View {
    @State var play = true
    
    var body: some View {
        VStack {
            Image(play ? "test1" : "test2")
                .resizable()
                .scaledToFit()
            PlayerButton(play: play) // 使用$符号进行引用的传递
            PlayerButton2(play: $play) // 使用$符号进行引用的传递
            Button {
                play.toggle()
            }label: {
                Text("外侧修改状态")
            }
            .font(.title)
        }
    }
}
// MARK: - ObservableObject + @Published + @StateObject + @ObservedObject
// 只有class才能实现ObservableObject的协议，struct不能
class Lover: ObservableObject {
    @Published var isLike = true
    @Published var age = 19
}
struct LoveView: View {
    // 对@State的多个属性集合的方式
    @StateObject var lover = Lover()
    @State var showBool = false
    var body: some View {
            VStack(spacing: 30) {
                HStack {
                    Spacer()

                    Image("test1")
                        .resizable()
                        .scaledToFill()
                        .frame(width: 200, height: 200)
                    .clipShape(Circle())
                    
                    Spacer()
                    
                    VStack(spacing: 50) {
                        Text("大美丽")
                        Text("年龄：\(lover.age)")
                        Image(systemName: lover.isLike ? "heart.fill" : "heart")
                            .font(.title)
                    }
                    Spacer()

                }
                
                Button {
                    lover.isLike.toggle()
                } label: {
                    Text(lover.isLike ? "取消赞" : "赞")
                }
                
                
                Button {
                    lover.age = lover.age + 1
                } label: {
                    Text("又涨了一岁")
                }

                Button {
                    showBool.toggle() // 是否跳转到下一个页面
                } label: {
                    Text("nextpage")
                // isPresented 传入 true 時，跳转到下一个页面，这是是$修饰的，说明里面也是@Binding修饰的
                }.sheet(isPresented: $showBool) {
                    // 这里传入的对象，有数对象本身已经是引用了，就不再需要$修饰了
                    LoveDetailView(lover: lover)
                }
            }
        }
}
struct LoveDetailView: View {
    // @Binding是针对一个简单属性
    // @ObservedObject针对的是一个ObservableObject类型的对象
    @ObservedObject var lover: Lover
    var body: some View {
            
            VStack {
                Text("芳年")
                    .font(.largeTitle)
                Image(systemName: "\(lover.age).circle")
                    .resizable()
                    .scaledToFit()
                    .frame(width: 200, height: 200)
                
                Stepper(value: $lover.age) {
                    Text("Age")
                }
                
                
            }
        }
}
// MARK: - ObservableObject + @Published + @StateObject + @EnvironmentObject
struct EnvironmentView: View {
    
    @StateObject var lover = Lover()
    @State var showBool = false
    
    var body: some View {
        VStack {
            Image(systemName: "\(lover.age).circle")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            Stepper(value: $lover.age) {
                Text("Age")
                
            }
            Button {
                showBool.toggle()
            } label: {
                Text("nextpage")
            // isPresented 傳入 true 時，跳轉一頁
            }.sheet(isPresented: $showBool) {
            // 跳轉 ShowDetail
                Page1View(lover: lover)
            }
            
        }
    }
    
}

struct Page1View: View {
    
    @ObservedObject var lover: Lover
    @State var showBool = false
    
    var body: some View {
        VStack {
            Text("Page1")
                .font(.largeTitle)
            Image(systemName: "\(lover.age).circle")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            
            Stepper(value: $lover.age) {
                Text("Age")
            }
            Button {
                showBool.toggle()
            } label: {
                Text("nextpage")
            // isPresented 傳入 true 時，跳轉一頁
            }.sheet(isPresented: $showBool) {
            // 跳轉 ShowDetail
                Page2View(lover: lover)
            }
            
        }
    }
}
struct Page2View: View {
    
    @ObservedObject var lover: Lover

    
    var body: some View {
        VStack {
            Text("Page2")
                .font(.largeTitle)
            Image(systemName: "\(lover.age).circle")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            
            Stepper(value: $lover.age) {
                Text("Age")
            }
            
            
        }
    }
}
struct EnvironmentView2: View {
    
    @StateObject var lover = Lover()
    @State var showBool = false
    
    var body: some View {
        VStack {
            Image(systemName: "\(lover.age).circle")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            // 如果内部是个引用传递，需要将对象的属性加上$才能是引用传递效果
            Stepper(value: $lover.age) {
                Text("Age")
                
            }
            Button {
                showBool.toggle()
            } label: {
                Text("nextpage")
            // isPresented 傳入 true 時，跳轉一頁
            }.sheet(isPresented: $showBool) {
            // 跳轉 ShowDetail
                Page1View2()
            }
            
        }
        .environmentObject(lover)
    }
    
}
struct Page1View2: View {
    
    @EnvironmentObject var lover: Lover
    @State var showBool = false
    
    var body: some View {
        VStack {
            Text("Page1")
                .font(.largeTitle)
            Image(systemName: "\(lover.age).circle")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            
            Stepper(value: $lover.age) {
                Text("Age")
            }
            Button {
                showBool.toggle()
            } label: {
                Text("nextpage")
            // isPresented 傳入 true 時，跳轉一頁
            }.sheet(isPresented: $showBool) {
            // 跳轉 ShowDetail
                Page2View2()
            }
            
        }
    }
}
struct Page2View2: View {
    
    @EnvironmentObject var lover: Lover

    
    var body: some View {
        VStack {
            Text("Page2")
                .font(.largeTitle)
            Image(systemName: "\(lover.age).circle")
                .resizable()
                .scaledToFit()
                .frame(width: 200, height: 200)
            
            Stepper(value: $lover.age) {
                Text("Age")
            }
            
            
        }
    }
}
// MARK: - 预览
struct MyContentViewdfdf: PreviewProvider {
    static var previews: some View {
        VStack {
//            MyContentView()
//            PlayerButton()
//            BindingView()
//            LoveView()
//            EnvironmentView(lover: Lover())
            EnvironmentView2(lover: Lover())
        }
    }
}

```