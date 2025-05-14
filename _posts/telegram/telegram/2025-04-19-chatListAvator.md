---
title: Telegram研究：最近会话记录的头像
author: 独孤流
date: 2025-04-19 14:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---


ChatListNodeEntries.swift
ChatListNode.swift

preparedChatListNodeViewTransition
ChatListNodeViewTransitionInsertEntry


ChatListNodeEntry -> ChatListNodeViewTransitionInsertEntry -> ListViewInsertItem -> ChatListItem

mappedInsertEntryies -> ListViewInsertItem -> item -> ChatListItem -> content -> .peer(peer+messages+...) + .groupReference(groupId+peers+message+...)

mappedUpdateEntryies -> ListViewUpdateItem -> item -> ChatListItem -> content 

ChatListItem -> 
nodeConfiguredForParams ->  + node.setupItem(item: self, synchronousLoads: synchronousLoads)
updateNode -> nodeValue.setupItem(item: self, synchronousLoads: false)

ChatListItemNode-> setupItem -> item.content -> avatarNode.setPeer -> contentNode.setPeer -> 

AvatarNode.StoryStats

public enum ChatListItemContent {
    case loading 
    case peer(PeerData)  // 单聊、群聊、频道、私密会话
    case groupReference(GroupReferenceData)
}

聊天会话里的会话类型，有4类
public enum EnginePeer: Equatable {
    case user(TelegramUser) // 单聊
    case legacyGroup(TelegramGroup) // 传统群
    case channel(TelegramChannel) // 频道
    case secretChat(TelegramSecretChat) // 加密聊天
}

用户类型
public protocol Peer: AnyObject, PostboxCoding {
    var id: PeerId { get }
    var indexName: PeerIndexNameRepresentation { get }
    var associatedPeerId: PeerId? { get }
    var notificationSettingsPeerId: PeerId? { get }
    var associatedMediaIds: [MediaId]? { get }
    var timeoutAttribute: UInt32? { get }
    
    func isEqual(_ other: Peer) -> Bool
}
public final class TelegramChannel: Peer, Equatable {}
public final class TelegramGroup: Peer, Equatable {}
public final class TelegramUser: Peer, Equatable {}
public final class TelegramSecretChat: Peer, Equatable {}


item.content -> .peer(peerData) -> .user(author) || peerData.peer.chatMainPeer -> peer.id
item.context.account.peerId