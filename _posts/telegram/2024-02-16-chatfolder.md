---
title: Telegram研：ItemListController使用之Chat Folders
author: 独孤流
date: 2024-02-16 18:04:00 +0800
categories: [Telegram]
tags: [Telegram, im]     # TAG names should always be lowercase
---

 
  - https://github.com/TelegramMessenger/Telegram-iOS  


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