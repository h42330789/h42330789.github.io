---
title: Telegram研究：联系人同步
author: 独孤流
date: 2025-06-19 10:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

初始化加载数据流程

`ContactsController.loadDisplayNode` -> `ContactsControllerNode.init` -> `ContactListNode.init`
`self.enableUpdates = true/false` -> `set` -> 值变化 -> `self.context.engine.data.subscribe`

`self.context.engine.data.subscribe` -> `EngineData` ->`_subscribe` -> `self.postbox.combinedView(keys: Array(keys))`   -> `Signal` -> `self.impl.with` -> `impl.combinedView` -> `transactionSignal` -> 

`postboxViewForKey` -> `.contacts(accountPeerId, includePresences)` -> `MutableContactPeersView(postbox: postbox, accountPeerId: accountPeerId, includePresences: includePresences)` -> `postbox.contactsTable.get()` + `postbox.peerTable.get(peerId)` + `postbox.peerPresenceTable.get(peerId)`

ContactPeersView
MutableContactPeersView
init(_ mutableView: MutableContactPeersView) {

// ContactsData.swift 第32行
return EngineContactList(peers: view.peers.map(EnginePeer.init), presences: view.peerPresences.mapValues(EnginePeer.Presence.init))

EngineContactList(peers: view.peers.map(EnginePeer.init), presences: view.peerPresences.mapValues(EnginePeer.Presence.init))

// ContactListNode.swift 第1798行
var peers = view.0.peers.map({ ContactListPeer.peer(peer: $0._asPeer(), isGlobal: false, participantCount: nil) })

// 排序
// ContactListNode
contactListNodeEntries
case let .orderedByPresence(options):
case let .natural(options, _, _):
orderedPeers = peers.sorted(by: { lhs, rhs in

// ContactListNode.swift 第1800-1820行
peers = peers.filter { contact in
    switch contact {
    case let .peer(peer, _, _):
        if requirePhoneNumbers, let user = peer as? TelegramUser {
            let phone = user.phone ?? ""
            if phone.isEmpty {
                return false
            }
        }
        return !existingPeerIds.contains(peer.id) && !pendingRemovalPeerIds.contains(peer.id)
    default:
        return true
    }
}

Peer的类型
public final class TelegramUser: Peer, Equatable
public final class TelegramSecretChat: Peer, Equatable
public final class TelegramGroup: Peer, Equatable
public final class TelegramChannel: Peer, Equatable

✅ 举例：有哪些类型实现了 Peer
TelegramUser: 普通用户；

TelegramGroup: 群组；

TelegramChannel: 频道、超级群；

TelegramSecretChat: 私密对话。

✅ 总结表
属性/方法	含义说明
id	当前 peer 的唯一标识（如 userId、groupId）
indexName	用于搜索、排序的名称表示
associatedPeerId	逻辑关联的另一个 peer，例如私密聊天、评论群等
notificationSettingsPeerId	使用哪个 peer 的通知设置（继承关系）
associatedMediaIds	头像、封面等关联媒体资源 ID
timeoutAttribute	超时属性，如 secret chat 或临时 peer 的超时时间戳
isEqual	判断两个 peer 是否在逻辑上相等

搜索数据流程
SearchBarNode.textFieldDidChange -> ContactsSearchContainerNode.searchTextUpdated ->  ContactsSearchContainerNode -> searchQuery.set
ContactsSearchContainerNode -> `foundLocalContacts = context.engine.contacts.searchContacts(query: query.lowercased())`
-> `self.account.postbox.searchContacts(query: query)`

self.impl.with -> impl.searchContacts
-> `stringIndexTokens(query, transliteration: .none)`
-> `self.peerNameIndexTable.matchingPeerIds`
-> `peerNameTokenIndexTable.matchingReferences` -> `contactTable.isContact(peerId: peerId)`

`self.peerTable.get(peerId)`
`self.peerPresenceTable.get(peerId)`
`contactPeers.sort`

SearchBarNode.textFieldDidChange -> ContactsSearchContainerNode.searchTextUpdated ->  ContactsSearchContainerNode -> searchQuery.set
ContactsSearchContainerNode -> context.engine.contacts.searchRemotePeers(query: query)
TelegramEngine -> _internal_searchPeers
network.request(Api.functions.contacts.search(q: query, limit: 20), automaticFloodWait: false)
updatePeers


修改好友备注
