---
title: Telegram研究：聊天列表Bubble里的每个类型的细分
author: 独孤流
date: 2025-04-16 14:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

ChatMessageBubbleItemNode: ChatMessageItem: ListViewItemNode

UI层级结\
ChatMessageBubbleItemNode\
-- ChatBubbleMainContainerNode\
--- ContextExtractedContentContainingNode\
---- ContextExtractedContentView
UIView\
---- ChatMessageBubbleClippingNode\
----- MessageBubbleContentNode

根据item变成BubbleContentNode
数据： ChatMessageItemImpl
`ChatMessageBubbbleItemNode.swift`
`ChatMessageBubbleItemNode` -> `asyncLayout` -> `beginLayout` -> `contentNodeMessagesAndClassesForItem` -> `ChatMessageItem > ... AnyClass ...` -> `for contentNodeItemValue in contentNodeMessagesAndClasses` -> `AnyClass` -> `contentNodeItem.type == xxx.self` -> `let contentNode = (contentNodeItem.type as! ChatMessageBubbleContentNode.Type).init()`

赋值
ChatMessageItemImpl.nodeConfiguredParams -> ChatMessageBubbleItemNode.asyncLayout -> ChatMessageBubbleItemNode.beginLayout -> ChatMessageTextBubbleContentNode.asynclayoutContent -> item.message.text

基类: MessageBubbleContentNode (系统消息也用是个)\
文本: ChatMessageTextBubbleContentNode （单独的文本或图文里的文本）\
图片: ChatMessageMediaBubbleContentNode （单独的图片、gif或混合多图文的图）\
。。。。

时间和状态：MessageDateAndStatusNode

动画（ChatMessageAnimatedStickerItemNode）\
基类：ManagedAnimationNode\
骰子：ManagedDiceAnimationNode\
老虎机slot: SlotMachineAnimationNode\
一般sticker：DefaultAnimatedStickerNodeImpl