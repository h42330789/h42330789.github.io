---
title: Telegram研究：头像获取流程
author: 独孤流
date: 2025-04-21 14:04:00 +0800
categories: [Telegram, Telegram及Demo]
tags: [Telegram, im]     # TAG names should always be lowercase
---

ChatListItemNode -> setupItem -> AvatarNode.setPeer -> AvatarNode.ContentNode.setPeer -> 

representation用户头像信息 PeerAvatar.peerAvatarImage -> peerAvatarImage -> peerAvatarImageData

根据id获取完整储存路径
-> postBox.mediaBox.resourceData -> MediaBox.resourceData(id: resource.id,...) -> 
storePathsForId(id) -> return ResourceStorePaths(partial: "\(self.basePath)/\(fileNameForId(id))_partial", complete: "\(self.basePath)/\(fileNameForId(id))")

如果文件存在，根据路径获取文件大小
FileSize.swift -> fileSize -> lstat(path, &value) -> try? Data(contentsOf: URL(fileURLWithPath: maybeData.path)) 

如果文件不存在，需要拉取文件

Display.ASImageNode： DisplayNode 一般静态图片
Display.ImageNode: DisplayNode 网络图片
Display.TransformImageNode: DisplayNode 动图
InstantPageUI.InstantPageImageNode: DisplayNode
Components.WallpaperGalleryScreen.BlurredImageNode: ASDisplayNode --> BlurView --> BlurLayer

动画
UniversalVideoNode -> NativeVideoContentNode -> MediaPlayerNode -> MediaPlayerNodeLaye -> AVSampleBufferDisplayLayerContentLayer