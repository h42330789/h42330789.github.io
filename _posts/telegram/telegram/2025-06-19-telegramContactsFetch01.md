---
title: Telegram研究：联系人同步
author: 独孤流
date: 2025-06-19 14:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

### 联系人相关操作和同步
- 1、拉取联系人列表（初始化、更新）
- 2、删除联系人
- 3、增加联系人
- 4、修改联系人昵称
- 5、联系人头像变化
- 6、搜索联系人

### 1、拉取联系人列表（初始化、更新）

##### 1.1、关键字
```
// 生成对比参数
`hashForCountAndIds`
// 到服务器对比
`updatedRemoteContactPeers` -> `Api.functions.contacts.getContacts`
// 手动导入的手机通讯录的人数
`transaction.replaceRemoteContactCount(totalCount)`
// 更新用户
`updatePeers(transaction: transaction, accountPeerId: accountPeerId, peers: AccumulatedPeers(users: partUsers))`
// 更新联系人id列表
`transaction.replaceContactPeerIds(updatedIds)`
```


AppDelegate.swift
SharedAccountContextImpl -> init -> account.resetStateManagement

Account.swift
restartContactManagement -> contactSyncManager.beginSync 

ContactSyncManager.swift
beginSync(importableContacts:xxx) -> self.impl.with -> ContactSyncManagerImpl -> beginSync(importableContacts:xxx) -> 
strongSelf.addOperation(.waitForUpdatedState)
strongSelf.addOperation(.updatePresences)
strongSelf.addOperation(.sync(importableContacts: importableContacts))

-> updateOperations -> startOperation 
-> 读取在线时间 -> .updatePresences -> updateContactPresences
-> 可引入的联系人 -> .sync(importableContacts) -> syncContactsOnce(network:...)


submodules/TelegramCore/Sources/TelegramEngine/Contacts/

ContactManagement.swift
`syncContactsOnce(network:...)` -> `postbox.transaction` -> 
本地表contactsTable获取联系人id列表: transaction.getContactPeerIds() -> scan -> 获取全部
本地表metadataTable获取联系人人数: transaction.getRemoteContactCount() -> get -> `.RemoteContactCount`

totalCount+peerIds -> 对id进行排序 -> `hashForCountAndIds` -> hash -> 去服务器对比 -> `updatedRemoteContactPeers` -> `Api.functions.contacts.getContacts(hash: hash)` -> FunctionDescription(name: "contacts.getContacts", parameters: [("hash",...)]) -> Api.parse -> 

没有变就不处理:  .contactsNotModified 
变化了就会返回人员、总数: .contacts(_, savedCount, users) 

-> appliedUpdatedPeers = updatedPeers -> `(peers, totalCount) = peersAndPresences`
-> `postbox.transaction`

更新总数 transaction.replaceRemoteContactCount(totalCount) -> postbox?.replaceRemoteContactCount
-> metadataTable.setRemoteContactCount -> valueBox.set
更新用户
如果本地没有数据，一般是刚卸载安装 -> 按500人分组 -> 每次都已有id与接口id进行并集 -> updatedIds.formUnion(partUsers.map { $0.peerId }) -> transaction.replaceContactPeerIds(updatedIds)
如果本地有数据。直接更新id -> transaction.replaceContactPeerIds(Set(peers.users.keys))

postbox?.replaceContactPeerIds -> contactsTable.replace

ContactTable -> beforeCommit -> 对比历史的数据、最新的数据 -> removedPeerIds + addedPeerIds -> 遍历删除 -> 遍历增加

override func beforeCommit() {
        if let peerIdsBeforeModification = self.peerIdsBeforeModification {
            if let peerIds = self.peerIds {
                let removedPeerIds = peerIdsBeforeModification.subtracting(peerIds)
                let addedPeerIds = peerIds.subtracting(peerIdsBeforeModification)
                
                let sharedKey = self.key(PeerId(0))
                
                for peerId in removedPeerIds {
                    self.valueBox.remove(self.table, key: self.key(peerId, sharedKey: sharedKey), secure: false)
                    self.peerNameIndexTable.setPeerCategoryState(peerId: peerId, category: [.contacts], includes: false)
                }
                
                for peerId in addedPeerIds {
                    self.valueBox.set(self.table, key: self.key(peerId, sharedKey: sharedKey), value: MemoryBuffer())
                    self.peerNameIndexTable.setPeerCategoryState(peerId: peerId, category: [.contacts], includes: true)
                }
            } else {
                assertionFailure()
            }
            
            self.peerIdsBeforeModification = nil
        }

        if !self.useCaches {
            self.peerIds = nil
        }
    }


syncContactsOnce -> updatedRemoteContactPeers -> Api.functions.contacts.getContacts(hash: hash)

只要开启事务处理
postbox.transaction -> 处理完事务里的内容业务逻辑 -> beginInternalTransaction -> internalTransaction
-> valueBox.begin() -> beforeCommit -> 跟新所有表的缓存 -> ... peerTable.transactionUpdatedPeers(contactsTable:...) ... -> valueBox.commit

// 更新用户信息
updatePeersCustom(transaction: transaction, peers: parsedPeers, update: { _, updated in updated })
updatePeersInternal -> postbox?.updatePeers -> 
self.peerTable.set(updatedPeer)
self.peerNameIndexTable.markPeerNameUpdated
self.peerNameIndexTable.markPeerNameUpdated

// 更新用户在线信息
updatePeerPresences(transaction: transaction, accountPeerId: accountPeerId, peerPresences: peers.users)

----
tg通过uid的排序然后combineInt64Hash组合hash，对比数据是否有变化

有些项目里，请求联系人接口时，会返回联系人的最后变化时间，下次同步数据时把时间最为参数，服务端对比，如果同步时间变化了就会有数据返回，然后全部保存这个更新时间作为下次请求时的参数

最后变更时间关心的是数据是否有变化，即使先加A，然后再移除A，实际数据没变化
实现方案1：每条操作都有独立的时间，且是逻辑删除，这样取最后一条时间比较，方便做增量更新
方案1随着时间的变化，记录的数据会很大，但是可以追溯有哪些变化

实现方案2：可以物理删除，但是有一个专门的表记录每个人的联系人列表的最后操作时间
方案2表里会有记录哪些新增，以及一条最后变更时间，数据量相对小些，但是没法追述变化了什么

tg的hash的方式是真实对比了联系人数据是否有了变化，tg类似方案2


