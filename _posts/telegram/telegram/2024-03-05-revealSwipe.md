---
title: Telegram研究：ListView/ItemListRevealOptionsItemNode侧滑删除
author: 独孤流
date: 2024-03-05 08:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

Telegram: \
仓库：[https://github.com/TelegramMessenger/Telegram-iOS](https://github.com/TelegramMessenger/Telegram-iOS)\
tag: `release-10.1`\
主要基类及协议：`ListView`、`ListViewInsertItem`、`ListViewItem`、`ItemListRevealOptionsItemNode`\
主要学习源码文件：`CallListViewTransition.swift`、`CallListControllerNode.swift`、`CallListCallItem.swift`\

本例Demo：
仓库： [https://github.com/h42330789/StudyIM/tree/feature/ListView/StudyAsynDisplay](https://github.com/h42330789/StudyIM/tree/feature/ListView/StudyAsynDisplay)\
分支：`origin/feature/ItemListController`\
主要Demo文件：`MySwipeDeleteListVC.swift`
> ### 前言
> `Telegram`包含了很多库，其中与UI有关的最核心的是`AsyncDisplayKit`、`Display`，其中UI库里使用最广的又是`ListView`,在列表里还有不少侧滑删除的功能

一、
1、要实现列表里展示测试的功能，itemNode要继承`ItemListRevealOptionsItemNode`
```
class CallListCallItemNode: ItemListRevealOptionsItemNode {
    // ...
}
```
2、设置滑动项`setRevealOptions(left: [xx,xx], right:[xx,xx])`\
活动内容的具体子内容：`ItemListRevealOptionsNode` -> `ItemListRevealOptionNode` -> `ASTextNode` + `ASImageNode` ，字体大小为固定的`Font.regular(17)或Font.medium(13)`
```
class CallListCallItemNode: ItemListRevealOptionsItemNode {
    // ...
    func asyncLayout() -> xxxx {
        // ...
        return { [weak self] .... in
            return (node, {[weak self] synchronousLoads in
                return (xxx, { [weak self] in 
                    // ...
                    // 设置左右可选项
                    self?.setRevealOptions((left: [], right: [ItemListRevealOption(key: 0, title: "删除", icon: .none, color: .green, textColor: .red)]))
                    // 设置初始化数据
                    self?.setRevealOptionsOpened(item.revealed, animated: animated)
                })
             })
        }
    }
}
```
3、子类处理滑动过程中的回调`updateRevealOffset(offset: CGFloat, transition: ContainedViewLayoutTransition)`
```
class CallListCallItemNode: ItemListRevealOptionsItemNode {
    // 侧滑功能滑动回调
    override func updateRevealOffset(offset: CGFloat, transition: ContainedViewLayoutTransition) {
        // ...
        // 重新计算每个组件的x，offset是总体移动的位置
        var avatarFrame = self.avatarNode.frame
        avatarFrame.origin.x = revealOffset + leftInset - 52.0
        transition.updateFrameAdditive(node: self.avatarNode, frame: avatarFrame)
        // ...
    }
    // 按钮展示回调
    override func revealOptionsInteractivelyOpened() {
        // ...
    }
    // 按钮关闭回调
    override func revealOptionsInteractivelyClosed() {
        // ...
    }
    // 点击某个选项，或一次性滑动结束只有一个时与点击一样效果的回调
    override func revealOptionSelected(_ option: ItemListRevealOption, animated: Bool) {
        self.setRevealOptionsOpened(false, animated: true)
        self.revealOptionsInteractivelyClosed()
        // 通过对option的内容，做删除执行等操作
        // ...
    }
}
```

6、执行删除操作
CallListControllerNode -> init -> callListNodeViewTransition [preparedCallListNodeViewTransition] -> enqueueTransition

CallListControllerNode -> init -> combineLatest(xxx)... mapToQueue{} -> preparedCallListNodeViewTransition -> `delete + insert + update` -> enqueueTransition -> dequeueTransition -> listNode.transaction
----

8、手势执行过程逻辑,在父类`ItemListRevealOptionsItemNode`里\
8.1、手势向左滑调用过程：\
`ItemListRevealOptionsItemNode` -> `didLoad` -> `view.addGestureRecognizer` -> `ItemListRevealOptionsGestureRecognizer:UIPanGestureRecognizer` -> `revealGesture(_:)` 

8.2、滑动手势的过程`revealGesture`：`.changed` -> `setupAndAddRightRevealNode没有时才添加` -> `updateRevealOffsetInternal` -> `updateRevealOffset(offset:,transition:)`

8.3、手势结束过程`revealGesture`：`.ended, .cancelled` -> `updateRevealOffsetInternal` -> `updateRevealOffset(offset:,transition:)`

8.4、`setupAndAddRightRevealNode(optionSelected:xx,tapticAction:xx)` -> `revealOptionSelected` -> 执行点击后的操作\
8.5、 `revealGesture(_:)` -> `.ended, .cancelled` -> `isDisplayingExtendedAction` -> `revealOptions.right.last` -> `revealOptionSelected` -> 执行点击后的操作