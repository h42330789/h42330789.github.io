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
`|>`: 拼接操作符\
调用方式：`let s2 = s1 |> xxx`， 等价于`clouse = xxx; let s2 = clouse(s1);`\
源码如下：

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
----

### combineLatest
#### `combineLatest`主要有以下几种变形
`combineLatest(queue:xxx, s1,s2,...)`:\
第一个参数是queue，可以为空, 后续的参数是信号，最少2个，最多可以有19个信号组合在一起

`combineLatest(queue:xxx, s1,t1,s2,t2)`: \
第一个参数是queue，将2个信息组合在一起，以及信号s1位置上的默认值t1，s2位置上的默认值t2

`combineLatest(queue:xxx, [s1,s2,...])`: \
组合任何信号的数组，信号数组的数量没有限制，但是信号里的值的类型需要相同，如果不同设置成`Any`就可以了，但一般都是signal特别多时才会调用

#### 实现原理：
内部调用：`combineLatestAny([xx,xx], combine:xxx,initialValues:[Int:Any],queue:xxx)`\
`signalOfAny`: \
将普通信号`Signal<T1, E>`转为`Signal<Any, E>`,当调用`Signal<Any, E>`的`anyxxx.start()`, 内部再调用原始信号的`xx.start()`进行值的传递

`initialValues`: \
`[Int:Any]`类型的字典, key是信号的组的index，value是信号产生的值，封装成`Atomic`进行管理，当这个值的数量与信号数量一致时就会触发调用观察着，这就是为什么`combineLatest(queue:xxx, s1,t1,s2,t2)`一开始就触发回调，这个也满足一开始初始值的就数量与信号数量一致

`combine`: \
这个是调用观察者回调时，将持有的`Atomic`里的值组合成一个新的值

`queue`: 信号所执行的队列

测试Demo如下：
```
func testSignalCombineLatest() {
    print("======testSignalCombineLatest======")
    /**
     combineLatest(queue:xxx, s1,s2,...): 第一个参数是queue，可以为空，信号，最少2个，最多可以有19个信号组合在一起
     combineLatest(queue:xxx, s1,t1,s2,t2): 第一个参数是queue，将2个信息组合在一起，还可以额外增加v1，v2
     combineLatest(queue:xxx, [s1,s2]]), 组合任何信号的数组
     内部会有一个Atomic，atomic里是一个[index:value]的字典，，用来存放signal里存值
     每次signal有值有都会存放在这里，这个state的数量与signal的数量一致时，没产生一个新的值都会触发
     */
    // 本质上都是调用的内部方法：combineLatestAny([xx,xx], combine:xxx,initialValues:[xx:xx],queue:xxx)，同时将signal转换为signalOfAny
    // signalOfAny本质是个中间信号，调用signalOfAny生成信号的start，就会触发原始信号的start，然后将原始信号的next，error，completed转发出来
    
    // ---- combineLatest(queue:xx, s1,v1,s2,v2)
    let s1 = Signal<String, NoError> { subcriber in
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.2, execute: {
            print("======s1-send======")
            subcriber.putNext("A")
        })
        return MetaDisposable()
    }
    let s2 = Signal<Int, NoError> { subcriber in
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.1, execute: {
            print("======s2-send======")
            subcriber.putNext(99)
        })
        return MetaDisposable()
    }
    // 有初始化值时，初始值会触发一次，任何一次修改都会触发一次
    let _ = combineLatest(s1, "B", s2, 100).start(next: { val1, val2 in
        print("======combineLatest--s1-s2--======")
        print(val1, val2)
    })
    // ---- combineLatest(queue:xx, s1,s2...), 最多可以传19个signal
    let s3 = Signal<String, NoError> { subcriber in
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.4, execute: {
            print("======s3-send======")
            subcriber.putNext("X")
        })
        return MetaDisposable()
    }
    let s4 = Signal<Int, NoError> { subcriber in
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.3, execute: {
            print("======s4-send======")
            subcriber.putNext(88)
        })
        return MetaDisposable()
    }
    // 没有初始值时，需要所有的signal都产生了值才会再combine里触发
    let _ = combineLatest(s3, s4).start(next: { val1, val2 in
        print("======combineLatest--s3-s4======")
        print(val1, val2)
    })
    // ---- combineLatest(queue:xx, [s1,s2...])
    // 需要signal里的类型值是一样的，要是不一样，只能设置类型为Any
    let s5 = Signal<Any, NoError> { subcriber in
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.4, execute: {
            print("======s5-send======")
            subcriber.putNext("aaa")
        })
        return MetaDisposable()
    }
    let s6 = Signal<Any, NoError> { subcriber in
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.3, execute: {
            print("======s6-send======")
            subcriber.putNext(111)
        })
        return MetaDisposable()
    }
    // 没有初始值时，需要所有的signal都产生了值才会再combine里触发
    let _ = combineLatest([s5, s6]).start(next: { valueList in
        print("======combineLatest--s5-s6======")
        print(valueList)
    })
    /**
    ======testSignalCombineLatest======
    ======combineLatest--s1-s2--======
    B 100
    ======s2-send======
    ======combineLatest--s1-s2--======
    B 99
    ======s1-send======
    ======combineLatest--s1-s2--======
    A 99
    ======s6-send======
    ======s4-send======
    ======s3-send======
    ======s5-send======
    ======combineLatest--s3-s4======
    X 88
    ======combineLatest--s5-s6======
    ["aaa", 111]
    */
}
```

----
### take
`take(count)`: \
配合`|>`操作符，将原始的信号执行count次数后就不再继续执行\
实现原理：`take(count)`会生成一个新的信号，新信号会对旧信号进行订阅，同时持有一个订阅次数的`Atomic`，当到达调用测试后就会执行`putCompletion`停止调用

源码如下：
```
// 输入一个count生成一个新的signal -> signal的信号
// public func take<T, E>(_ count: Int) -> ((Signal<T, E>) -> Signal<T, E>) {xxxx}
// s1 |> take(2) 就会变成 let cloure = take(2) let s2 = clouse(s1)
// take(xxx)里会把count存入一个Atomic，每次接受原始信号s1时这个值会+1，直到值与count相等时，调用putCompletion,停止调用
public func take<T, E>(_ count: Int) -> (Signal<T, E>) -> Signal<T, E> {
    return { signal in
        return Signal { subscriber in
            let counter = Atomic(value: 0)
            return signal.start(next: { next in
                var passthrough = false
                var complete = false
                let _ = counter.modify { value in
                    let updatedCount = value + 1
                    passthrough = updatedCount <= count
                    complete = updatedCount == count
                    return updatedCount
                }
                if passthrough {
                    subscriber.putNext(next)
                }
                if complete {
                    subscriber.putCompletion()
                }
            }, error: { error in
                subscriber.putError(error)
            }, completed: {
                subscriber.putCompletion()
            })
        }
    }
}
```
调用Demo如下：
```
func testSignalTake() {
    print("======testSignalTake======")
    // 输入一个count生成一个新的signal -> signal的信号
    // public func take<T, E>(_ count: Int) -> ((Signal<T, E>) -> Signal<T, E>) {xxxx}
    // s1 |> take(2) 就会变成 let cloure = take(2) let s2 = clouse(s1)
    // take(xxx)里会把count存入一个Atomic，每次接受原始信号s1时这个值会+1，直到值与count相等时，调用putCompletion,停止调用
    let s1 = Signal<String, NoError> { subcriber in
        subcriber.putNext("aaa")
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.2, execute: {
            print("======s1-send======")
            subcriber.putNext("bb")
        })
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.1, execute: {
            print("======s1-send======")
            subcriber.putNext("cc")
        })
        return MetaDisposable()
    }
    let s2 = s1 |> take(2)
    let _ = s2.start(next: { val in
        print("s2-s1-take: \(val)")
    })
    
    let s3 = Signal<Int, NoError> { subcriber in
        subcriber.putNext(111)
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.2, execute: {
            print("======s3-send======")
            subcriber.putNext(222)
        })
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.1, execute: {
            print("======s3-send======")
            subcriber.putNext(333)
        })
        return MetaDisposable()
    }
    let cloure: ((Signal<Int, NoError>) -> Signal<Int, NoError>) = take(1)
    let s4 = cloure(s3)
    let _ = s4.start(next: { val in
        print("s4-s3-take: \(val)")
    })
    // take(until:xxx)
    let s5 = Signal<String, NoError> { subcriber in
        subcriber.putNext("xxx")
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.2, execute: {
            print("======s5-send======")
            subcriber.putNext("yyy")
        })
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.1, execute: {
            print("======s5-send======")
            subcriber.putNext("zzz")
        })
        return MetaDisposable()
    }
    let s6 = s5 |> take(until: { val in
        // passthrough: 为true时都会触发next，complete为true时，以后就都不再执行next
        return SignalTakeAction(passthrough: val != "zzz", complete: val == "zzz")
    })
    let _ = s6.start(next: { val in
        print("s6-s5-take: \(val)")
    })
    /**
     ======testSignalTake======
     s2-s1-take: aaa
     s4-s3-take: 111
     s5-s1-take: xxx
     ======s1-send======
     ======s3-send======
     s2-s1-take: cc
     ======s5-send======
     ======s1-send======
     ======s5-send======
     ======s3-send======
     */
}
```
----
### single == take(1)

#### Timing
`delay`: 使用方式：`s1 |> delay(xxx, queue:xxqueue)` ， 延迟xxx秒后后再执行\
`timeout`: 使用方式：`s1 |> timeout(xxx, queue:xxqueue, altername: xxx)` ， 延迟xxx秒后后再执行,s1执行了就不执行altername\
`suspendAwareDelay`: 使用方式：`s1 |> suspendAwareDelay(xxx, granularity:xxx, queue:xxqueue)` ， 延迟xxx秒后后再执行


---
### 

### ValuePipe
### Timer

`Catch`: 
`restart`:
`recurse`:
`retry`:
`restartIfError`: