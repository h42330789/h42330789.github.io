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
`set(xxxSignal)`: 用来设置信号替换`value`,同时触发`next`\
`get()`: 用来增加一个订阅者信号，通过该信号的`start()`获取Promise里的`value`\

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