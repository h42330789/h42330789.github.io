---
title: Telegram研究：群列表同步
author: 独孤流
date: 2025-06-19 14:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

🔁 所有群聊、单聊、频道都复用这个 auth_key
|聊天类型	 |  加密方式	|是否使用独立密钥	|使用哪个 key 加密|
|----------|-------------|-----------------|--------------------|
|Secret Chat|	E2EE (AES-IGE + DH)	|✅ 每个会话独立 key	|每个 Secret Chat 的专属密钥|
|普通单聊	|MTProto 加密	|❌ 不独立	|客户端和服务器之间的 auth_key|
|群聊	|MTProto 加密	|❌ 不独立	|同上|
|频道	|MTProto 加密	|❌ 不独立	|同上|

每位成员在群里有一个自己的 Sender Key Record，包含：
一个对称密钥 sender_chain_key（AES）

一个消息计数器（防重放）

一个身份标识（发件人 ID）

🛠️ Signal 群消息加密流程（简化版）：
📦 第一步：分发 Sender Key（只分发一次）
每个成员第一次发送群消息时，生成一对：

sender_key_id（唯一标识）

sender_chain_key（对称加密 key）

这个 sender_chain_key 会被：

用群内每个接收者的 公钥（Curve25519） 分别加密

并作为“SenderKeyDistributionMessage” 发给群内每个成员

✉️ 第二步：消息加密（之后的每条消息）
发消息者用自己的 sender_chain_key + message_counter

使用 AES-GCM 加密消息内容

加密后的消息包含：

发送者 ID

消息计数器

加密内容

📥 第三步：消息接收与解密
每个接收方：

查自己的本地缓存，是否已经接收到过该发送者的 sender_key

若有，则使用该 key + 计数器 解密消息

若没有，会拒绝消息或回请求重发 key 分发消息

你设计的方式：wrapped_key = f(K_s, sk_sender, pk_receiver)

还是 Signal 的方式：wrapped_key = encrypt(K_s, pk_receiver)

✅ 情况 1：AES 密钥不变
🟢 新成员加入 → 可解历史消息
🔴 离开成员 → 还能继续解后续消息（安全隐患）

✅ 情况 2：AES 密钥更新
🔒 离开成员失效 → 无法再解新消息
🔒 新加入成员无历史密钥 → 无法解密过去消息（功能缺失）

🔁 这就是端对端群加密的经典「安全性 vs 可用性」权衡问题：
目标	描述	会牺牲什么
🔐 前向保密（Forward Secrecy）	离开的人立刻失效	❌ 新成员无法读历史
📚 历史可访问性（History Access）	新成员能读旧消息	❌ 离开的人也能读新消息
🧊 服务端透明（Server Blind）	服务端不知道 key	❌ 无法“补发密钥”实现兼容

🧠 那有没有「折中方案」？
是有的，不过都需要设计层面的「策略化授权」，比如下面这些：

🧩 方案 A：历史消息用不同 key + 多 key 管理
每隔一段时间/每 N 条消息 → 生成一个新的 K_i

每条消息记录 K_i 的 ID

新成员加入时，老成员（或群主）决定是否“转发”某些历史 key

转发使用新成员公钥封装

最后总结
群加密中：

若 秘钥保持不变：功能友好（新成员能看历史），但安全脆弱（老成员滥用）

若 秘钥经常更换：更安全（前向保密），但牺牲体验（历史消息无法访问）

你说的两句本质上就是这个悖论的最佳表述：

✅ 「秘钥不换 → 功能强但不安全」
✅ 「秘钥一换 → 安全了，但失去了历史访问性」


PeerTable (表ID: 2) - 存储用户、群组、频道信息
SecretChatTable - 存储秘密聊天信息
SecretChatStateTable - 存储秘密聊天状态和密钥
MessageHistoryTable - 存储消息历史
ContactTable - 存储联系人信息

5. 总结
Telegram中公钥存储的正确机制：
存储位置：PeerChatStateTable (表ID: 13)
数据结构：SecretChatState 包含 SecretChatKeychain
密钥数据：SecretChatKey 包含实际的密钥字节数据
访问方式：通过 transaction.getPeerChatState(peerId) 获取
持久化：所有数据都存储在SQLite数据库中，使用PostboxCoding协议进行序列化