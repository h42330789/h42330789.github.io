---
title: Telegram研究：群列表同步
author: 独孤流
date: 2025-06-19 14:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

TelegramCore/Account.swift
accountWithId(accountManager:...)
init(accountManager:...)

TelegramCore/ManagedServiceViews.swift
managedServiceViews(accountPeerId:...)

TelegramCore/ManagedChatListHoles.swift
managedChatListHoles(network:...)

TelegramCore/Holes.swift
fetchChatListHole(postbox:...)

TelegramCore/FetchChatList.swift
fetchChatList(accountPeerId:...)

TelegramCore/FetchChatList.swift
let requestChats = network.request(Api.functions.messages.getDialogs(flags: flags, folderId: requestFolderId, offsetDate: timestamp, offsetId: id, offsetPeer: peer, limit: limit, hash: hash))

TelegramApi/Api38.swift
getDialogs -> FunctionDescription(name: "messages.getDialogs",...)

let requestFeed = network.request(Api.functions.messages.getDialogs(flags: flags, folderId: groupId.rawValue, offsetDate: 0, offsetId: 0, offsetPeer: .inputPeerEmpty, limit: 32, hash: 0))


----

func firstHole() -> (PeerGroupId, ChatListHole)? {
    return self.sampledState.hole  // 返回检测到的空洞
}
updateTrackedChatListHoles


let request = network.request(Api.functions.updates.getDifference(flags...))


return network.request(Api.functions.updates.getState())

let request = network.request(Api.functions.updates.getDifference..)

// 1. 从本地数据库读取当前state
let currentState = transaction.getState() as? AuthorizedAccountState
// 得到：pts=100, qts=50, date=1640995200, seq=10

// 2. 调用updates.getState()获取服务器最新state
let serverState = network.request(Api.functions.updates.getState())
// 服务器返回：pts=150, qts=60, date=1640995800, seq=15

// 3. 使用本地state调用updates.getDifference()
let difference = network.request(Api.functions.updates.getDifference(
    pts: 100,    // 本地pts
    date: 1640995200,  // 本地date
    qts: 50      // 本地qts
))
// 服务器返回：pts=100到150之间的所有更新