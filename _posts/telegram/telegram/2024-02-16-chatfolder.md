---
title: Telegram研究：ItemListController使用之Chat Folders
author: 独孤流
date: 2024-02-16 18:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

 
Telegram: \
仓库：[https://github.com/TelegramMessenger/Telegram-iOS](https://github.com/TelegramMessenger/Telegram-iOS)\
tag: `release-10.1`\
主要基类及协议：`ItemListNodeEntry`、`ListViewItem`+`ItemListItem`、`ItemListRevealOptionsItemNode`+`ListViewItemNode`\
主要学习源码文件：`ChatListFilterPresetListController.swift`、`ChatListFilterPresetController.swift`\
主要业务类：
- `ChatListFilterPresetListEntry`、`ChatListFilterPresetEntry`: 原始的数据
- `ChatListFilterSettingsHeaderItem` + `ItemListSectionHeaderItem` + `ChatListFilterPresetListSuggestedItem`: 继承自`ListViewItem, ItemListItem`，设置对应的展示Node，处理点击回调
- `ChatListFilterSettingsHeaderItemNode` + `CallListGroupCallItemNode` + `CallListHoleItemNode`: 继承自`ItemListRevealOptionsItemNode->ListViewItemNode`，设置Node的展示样式及做布局大小计算
- `ItemListRevealOptionsItemNode`: 这个继承自`ListViewItemNode`, 这个类定义了可以侧滑删除相关的交互

本例Demo：
仓库： [https://github.com/h42330789/StudyIM/tree/feature/ListView/StudyAsynDisplay](https://github.com/h42330789/StudyIM/tree/feature/ListView/StudyAsynDisplay)\
分支：`origin/feature/ItemListController`\
主要Demo文件：`MyChatFolderVC.swift`、`EditChatFolderVC.swift`


![image](/assets/img/telegram/telegram_chatFolders_01.png)

PeerInfoScreen.swift → PeerInfoScreenNode → openSettings → .chatFolders →

ChatListFilterPresetListController.swift → chatListFilterPresetListController(context: ...) 

1、创建参数和回调 signal

signal.map(vcStatem (listState, arg)) → chatListFilterPresetListControllerEntries(presentationData:...) → 配置数据源 → [ChatListFilterPresetListEntry] → .screenHeader + .listHeader + .addItem + ...



SIgnal需要配置的内容：

1.1 controller相关信息

ItemListControllerState(title: 标题，leftNavBtn, rightNavBtn, backNabBtn, ...)

1.2 列表数据源信息

ItemListNodeState(entries: 数据源列表) → [ItemListNodeEntry] → chatListFilterPresetListControllerEntries

ChatListFilterPresetListEntry

1.2.1 需要实现 Identifiable

statbleId,

1.2.2 Comparable协议：

sortId

func <,

1.2.3： 配置Item → ListViewItem

ChatListFilterPresetListEntry → item(presentationData: ...) → .screenHeader → ChatListFilterSettingsHeaderItem → nodeConfiguredForParams → ChatListFilterSettingsHeaderItemNode → node.asyncLayout()(...)

ChatListFilterPresetListEntry → item(presentationData: ...) → .listHeader → ItemListSectionHeaderItem → nodeConfiguredForParams → ChatListFilterSettingsHeaderItemNode