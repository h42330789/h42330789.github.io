---
title: SwiftSignalKit研究一：基本概念及使用
author: 独孤流
date: 2024-02-22 21:04:00 +0800
categories: [Telegram, SwiftSignalKit]
tags: [Telegram, SwiftSignalKit]     # TAG names should always be lowercase
---

参考：
- [https://hubo.dev/2020-05-11-source-code-walkthrough-of-telegram-ios-part-2](https://hubo.dev/2020-05-11-source-code-walkthrough-of-telegram-ios-part-2/)
- [Telegram-iOS 源码分析：第二部分（SSignalKit）](https://www.jianshu.com/p/887de98ae9f2)
- [RxSwift核心](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core.html)

与Rxswif类似，主要包含的内容：`Signal`,`Subscriber`，`Disposable`

### Disposable
`Disposable`： 一个协议，包含有`func dispose()`的方法\
`StrictDisposable: Disposable`：通过`xxdispose.strict()`获取\
`_EmptyDisposable: Disposable`: 直接使用`EmptyDisposable`获取全局同一个\
`ActionDisposable: Disposable`: 在`dispose`或`deinit`时会执行一个action的block回调\
`MetaDisposable: Disposable`: 内部包含一个dispose，并且可以替换\
`DisposableSet: Disposable`: 内部包含一组使用Array管理的dispose集合，并且添加或删除新的dispo，`dispose()`时会调用所有子dispose对象的`dispose()`方法\
`DisposableDict: Disposable`: 内部包含使用Dict管理的dispose集合，并且通过Key设置dispose，`dispose()`时会调用所有子dispose对象的`dispose()`方法

### Subscriber
`Subscriber`: \
有`next`,`error`,`completed`回调\
有`pubNext`,`putError`,`putCompletion`方法

### Signal
`Signal`: 信号，只有一个`start(xxx)`方法，只要执行start，就会生成一个`subscriber`，同时调用`generator`回调
Signal创建方式一，使用`generator`创建: 
```
 Signal { subscriber in
        // ... 义务处理
        return xxxDispose
    }
```
其他默认方式
`Signal.single(xxx)`\
`Signal.complete()`\
`Signal.fail(xxx)`\
`Signal.never()`


### Promise
`Promise`: \
内部有一个存放数据的`value`, 订阅者列表`subscribers`\
`set(xxxSignal)`: 用来设置信号替换`value`,同时触发`xxxSignal`的generator回调\
`get()`: 用来增加一个订阅者信号，通过该信号的`start()`获取Promise里的`value`
```
 func testPromise1() {
    print("======testPromise1======")
    
    let promise1 = Promise<String>()
    // 由于第一次获取时，没有值，不会触发回调
    var promise1Value: String? = nil
    let dispose0 = promise1.get().start(next: { list in
        print("\(Date()) dispose0 get1--\(list)")
        promise1Value = list
    })
    dispose0.dispose()
    let promise1Value2 = promise1.rawValue
    print("\(Date()) promise1Value--\(promise1Value ?? "nil") promise1Value2:\(promise1Value2 ?? "nil")")
    // 由于第一次获取时，没有值，不会触发回调
    let dispose1 = promise1.get().start(next: { list in
        print("\(Date()) dispose1 get1--\(list)")
    })
    // 设置Signal后，会立即执行signal的generator, 通过next拿到generatoer里设置的putNext()的值
    // 获取到值后，会回调执行所有get()创建的subscriber
    promise1.set(Signal<String, NoError> { subscriber in
        // 会触发此刻已经存在的所有get()创建的所有监听者
        print("\(Date()) 设置值--aaa")
        subscriber.putNext("aaaa")
        DispatchQueue.global().asyncAfter(deadline: .now() + 2, execute: {
            // 会触发此刻已经存在的所有get()创建的所有监听者
            print("\(Date()) 设置值--bbb")
            subscriber.putNext("bbb")
        })
        return MetaDisposable()
    })
    // 由于promise里已经有值了，直接获取存在的值
    let dispose2 = promise1.get().start(next: { list in
        print("\(Date()) dispose2 get2--\(list)")
    })
    let promise1Value3 = promise1.rawValue
    print("\(Date()) promise1Value3:\(promise1Value3 ?? "nil")")
    /**
     ======testPromise1======
     2024-02-24 08:23:38 +0000 promise1Value--nil promise1Value2:nil
     2024-02-24 08:23:38 +0000 设置值--aaa
     2024-02-24 08:23:38 +0000 dispose1 get1--aaaa
     2024-02-24 08:23:38 +0000 dispose2 get2--aaaa
     2024-02-24 08:23:38 +0000 promise1Value3:aaaa
     2024-02-24 08:23:40 +0000 设置值--bbb
     2024-02-24 08:23:40 +0000 dispose1 get1--bbb
     2024-02-24 08:23:40 +0000 dispose2 get2--bbb
     */
}
// 使用使用完立即销毁的方式获取默认值
extension Promise {
    var rawValue: T? {
        get {
            // 通过get获取signal，调用start()获取最新的值并添加subscriber
            var valueObj: T? = nil
            let valueDispose = self.get().start { value in
                // 如果当前值为空时不会走这个回调，如果值不为空时会走这个回调
                valueObj = value
            }
            // 销毁了dispose，后续value值变化也不会触发上面的赋值操作
            valueDispose.dispose()
            // 返回获取的值
            return valueObj
        } set {
            if let val = newValue {
                self.set(Signal {subscriber in
                    subscriber.putNext(val)
                    return EmptyDisposable
                })
            }
        }
    }
}
```
利用Promise的默认值为nil，异步设置的特点特别适合网络请求的场景
1、进入一个页面后立即开始网络请求数据，网络请求完成后设置值并刷新页面
```
func testPromise2() {
    print("======testPromise2======")
    // 这里可以设置默认值，如果有的话
    let networkPromise = Promise<String>()
    networkPromise.rawValue = UserDefaults.standard.string(forKey: "net1")
   
    // 设置Signal后，会立即执行signal的generator, 通过next拿到generatoer里设置的putNext()的值
    // 获取到值后，会回调执行所有get()创建的subscriber
    networkPromise.set(Signal<String, NoError> { subscriber in
        guard let url = URL(string: "https://reqres.in/api/users/2") else {
            subscriber.putCompletion()
            return EmptyDisposable
        }
        let request = URLRequest(url: url)
        let task = URLSession.shared.dataTask(with: request, completionHandler: {data, response, error in
            if let data = data {
                // 数据请求成功
                var respStr = String(data: data, encoding: .utf8) ?? "--"
                respStr = "response--Data"
                UserDefaults.standard.setValue(respStr, forKey: "net1")
                print("网络请求成功----")
                subscriber.putNext(respStr)
            } else {
                // 如果有错误
                subscriber.putCompletion()
            }
        })
        // 开始请求
        task.resume()
        // 发起网络请求
        return ActionDisposable {
            // 如果销毁了，可以将网络请求取消
            task.cancel()
        }
    })
    // 监听数据，有数据变化才刷新
    let uiDispose = (networkPromise.get() |> distinctUntilChanged
    ).start(next: { str in
        // 网络请求后有数据变化，更新UI
        print("更新UI: \(str)")
    }, completed: {
        print("更新UI: completed")
    })
}
```



`ValuePromise`: \
内部有一个存放数据的`value`, 订阅者列表`subscribers`\
`set(xxx)`: 用来设置新值替换`value`,同时触发所有的`subscribers`的`next`\
`get()`: 用来增加一个订阅者信号,并在`start()`时加入到`subscribers`
```
func testValuePromise1() {
    print("======testValuePromise1======")
    let promise1 = ValuePromise("testA", ignoreRepeated: true)
    var promise1Value: String = "nil"
    print("promise1Value: \(promise1Value)")
    let promise1Dispose1 = promise1.get().start { value in
        print("promise1Dispose1-next:\(value)")
        promise1Value = value
    }
    // promise1Dispose1没在start时，会获取最新的值，所以promise1Value会变成最新的 testA
    print("promise1Value: \(promise1Value)")
    // 设置新值,这里由于promise1Dispose1还没有销毁，这里的set也会触发收到值
    promise1.set(promise1Value + "---" + "testB")
    
    // 重新获取，在start时会获取最新的值 testA---testB
    let promise1Dispose2 = promise1.get().start { value in
        print("promise1Dispose2-next: \(value)")
    }
    // promise1Dispose1没有销毁，会最新的设置也会生效，所以promise1Value会变成最新的 testA---testB
    print("promise1Value: \(promise1Value)")
    promise1Dispose1.dispose()
    promise1Dispose2.dispose()
    /**
        ======testValuePromise1======
        promise1Value: nil
        promise1Dispose1-next:testA
        promise1Value: testA
        promise1Dispose1-next:testA---testB
        promise1Dispose2-next: testA---testB
        promise1Value: testA---testB
        */
}
    
func testValuePromise2() {
    print("======testValuePromise2======")
    let promise2 = ValuePromise("testX", ignoreRepeated: true)
    var promise2Value: String = "nil"
    print("promise2Value: \(promise2Value)")
    let promise2Dispose1 = promise2.get().start { value in
        print("promise2Dispose1-next:\(value)")
        promise2Value = value
        
    }
    // 销毁了dispose，promise2Dispose1对应的subscriber也会被销毁，后续再给promise2设置时，promise2Dispose1再也不会处理了
    promise2Dispose1.dispose()
    // promise2Dispose1没在start时，会获取最新的值，所以promise2Value会变成最新的 testX
    print("promise2Value: \(promise2Value)")
    
    // 设置新值,这里由于promise1Dispose1还没有销毁，这里的set不会触发被销毁的subscriber
    promise2.set(promise2Value + "---" + "testY")
    
    // 重新获取，在start时会获取最新的值 test1---testB
    let promise2Dispose2 = promise2.get().start { value in
        print("promise2Dispose2-next: \(value)")
    }
    // 所以promise2Value会变成最新的 testX
    print("promise2Value: \(promise2Value)")
    promise2Dispose2.dispose()
    /**
        ======testValuePromise2======
        promise2Value: nil
        promise2Dispose1-next:testX
        promise2Value: testX
        promise2Dispose2-next: testX---testY
        promise2Value: testX
        */
}
```
使用ValuePromise的特性，可以增加扩展方法直接获取不可以访问的值
```
extension ValuePromise {
    var rawValue: T {
        // 通过get获取signal，调用start()获取最新的值并添加subscriber
        var valueObj: T!
        let valueDispose = self.get().start { value in
            valueObj = value
            
        }
        // 销毁了dispose，将valueDispose对应的subscriber也会被销毁
        valueDispose.dispose()
        // 返回获取的值
        return valueObj
    }
}
func testValuePromise3() {
    print("======testValuePromise3======")
    let promise3 = ValuePromise("aaa", ignoreRepeated: true)
    let promise3Value = promise3.rawValue
    print("promise3Value: \(promise3Value)")
    
    promise3.set(promise3Value + "---" + "bbb")
    
    // 重新获取，在start时会获取最新的值 test1---testB
    let promise3Dispose2 = promise3.get().start { value in
        print("promise3Dispose2-next: \(value)")
    }
    // 所以promise2Value会变成最新的 testX
    print("promise3Value: \(promise3Value)")
    promise3Dispose2.dispose()
    /**
        ======testValuePromise3======
        promise3Value: aaa
        promise3Dispose2-next: aaa---bbb
        promise3Value: aaa
        */
}
```

### Bag
`Bag`：跟数组Array的功能差不多，有`get`，`add`，`remove`等功能\
`SparseBag: Sequence`: 同样是组数的基本功能
`CounterBag`: 用于维护一个计数的类，没什么特别的功能


### `Atomic`
一个普通的对数据持有的封装，在线程安全的方式修改数据，可以对持有的数据进行转换主要方法：\
`with`: 将数据转换为其他内容并返回转换后的类型，并不修改自己持有的数据，类似`map`\
`modify`: 将数据转换为同类型的其他数据，并将转换好的数据替换自己的数据，同时返回替换后的数据,比`map`多了修改持有的数据\
`swap`: 将同类型的新数据替换旧数据，同时返回旧数据