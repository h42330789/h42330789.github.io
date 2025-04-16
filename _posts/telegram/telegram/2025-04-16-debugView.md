---
title: Telegram研究：使用viewClass或setViewBlock自定义view方便debug定位
author: 独孤流
date: 2025-04-16 09:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

> ### 前言
> 由于Telegram的代码量太大，而且显示的几乎全是`_ASDisplayView`，很多node文件有4000~5000行代码，导致调试查看十分麻烦，`AsyncDisplayKit`的`ASDisplayNode`提供了设置自定义类的方法，可以在debug时方便定位问题

`setViewBlock`的优先级要高于`viewClass`

#### 方案一、在`ASDisplayNode`子类里设置`setViewBlock`
例子
```
private final class ChatControllerNodeView: UITracingLayerView, WindowInputAccessoryHeightProvider {

.....

}

class ChatControllerNode: ASDisplayNode, ASScrollViewDelegate {

         init(context: ......) {

            .........

         // 提在ASDisplayNode的子类里设置setViewBloc，返回一个UIView的子类实例即可

          self.setViewBlock({

                  return ChatControllerNodeView()

         })

         ..........

        }

}
```


`ListView`都通过`setViewBlock`默认设置了`ListViewBackingView`\
源码如下：
```
open class ListView: ASDisplayNode, ASScrollViewDelegate, ASGestureRecognizerDelegate {

        override public init() {

              // ...
                 self.setViewBlock({ () -> UIView in

                    return ListViewBackingView()

               })

              // ...

       }

}
```

#### 方案二、重写ASDisplayNode子类的viewClass方法
例子：
```
public class ChatMessageBubbleItemNode: ChatMessageItemView, ChatMessagePreviewItemNode {

    #if DEBUG
    // 提供一个_ASDisplayView的子类即可
    class ChatMessageBubbleItemNodeView: _ASDisplayView { }

    // 提在ASDisplayNode的子类里重写viewClass方法，返回个_ASDisplayView的子类的类型即可
    override public class func viewClass() -> AnyClass {
        return ChatMessageBubbleItemNodeView.self
    }
    #endif

    // ...
}
```