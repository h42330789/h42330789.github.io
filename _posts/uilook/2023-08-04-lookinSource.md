---
title: LookIn源码研究与学习
author: 独孤流
date: 2023-08-04 23:12:00 +0800
categories: [UI查看]
tags: [lookin]     # TAG names should always be lowercase
---

源码地址：
- iOS 端 LookinServer：https://github.com/QMUI/LookinServer
- macOS 端软件：https://github.com/hughkli/Lookin/
- [Lookin原理及5个开发难点](https://github.com/hughkli/Lookin/blob/Develop/Docs/Lookin%E5%8E%9F%E7%90%86%E5%8F%8A5%E4%B8%AA%E5%BC%80%E5%8F%91%E9%9A%BE%E7%82%B9.md)
- https://github.com/hughkli/KKConnector
- https://medium.com/better-programming/a-side-by-side-comparison-of-two-great-ios-views-debugging-tools-85fefbf69881
- https://github.com/FLEXTool/FLEX

技巧
  
如何在 Lookin 中展示自定义信息: https://bytedance.larkoffice.com/docx/TRridRXeUoErMTxs94bcnGchnlb
如何在 Lookin 中展示更多成员变量: https://bytedance.larkoffice.com/docx/CKRndHqdeoub11xSqUZcMlFhnWe
如何为 Lookin 开启 Swift 优化: https://bytedance.larkoffice.com/docx/GFRLdzpeKoakeyxvwgCcZ5XdnTb
文档汇总：https://bytedance.larkoffice.com/docx/Yvv1d57XQoe5l0xZ0ZRc0ILfnWb

项目结构如下：
```
Src
├── Base
├── Main
│   ├── Server
│   │   ├── Category
│   │   ├── Connection
│   │   │   └── RequestHandler
│   │   └── Others
│   └── Shared
│       ├── Category
│       └── Peertalk
└── Swift
```

### 一、监听、读写、业务主要类
管理类型：`Server.Connection.LKS_ConnectionManager`\
读写io类：`Peertalk.Lookin_PTChannel`\
处理业务类：`Server.Connection.LKS_RequestHandler`

#### 1、启动服务
利用所有类都会在App启动时调用`load`方法进行启动:\
`LKS_ConnectionManager` -> `+ (void)load` -> `sharedInstance` -> `init` -> 注册通知`UIApplicationDidBecomeActiveNotification`\
 -> `_handleApplicationDidBecomeActive` -> `_tryToListenOnPortFrom:to:current:` \
 -> `Lookin_PTChannel` -> `channel listenOnPort:IPv4Address:callback:` -> 系统的`bind`、`close`、`listen`等方法

 链接端口逻辑：
 递归尝试from-to之间的端口，如果listen失败，则端口号+1进行监听
 ```
     [channel listenOnPort:currentPort IPv4Address:INADDR_LOOPBACK callback:^(NSError *error) {
        if (error) {
            if (error.code == 48) {
                // 该地址已被占用
            } else {
                // 未知失败
            }
            
            if (currentPort < toPort) {
                // 尝试下一个端口
                NSLog(@"LookinServer - 127.0.0.1:%d is unavailable(%@). Will try anothor address ...", currentPort, error);
                [self _tryToListenOnPortFrom:fromPort to:toPort current:(currentPort + 1)];
            } else {
                // 所有端口都尝试完毕，全部失败
                NSLog(@"LookinServer - 127.0.0.1:%d is unavailable(%@).", currentPort, error);
                NSLog(@"LookinServer - Connect failed in the end. Ask for help: lookin@lookin.work");
            }
            
        } else {
            // 成功
            NSLog(@"LookinServer - Connected successfully on 127.0.0.1:%d", currentPort);
            // 此时 peerChannel_ 状态为 listening
            self.peerChannel_ = channel;
        }
    }];
 ```

#### 2、接收数据
 `listenOnPort:` -> `while` ->` acceptIncomingConnection` -> `type` + `tag` + `data`
 -> `LKS_RequestHandler` ->  `canHandleRequestType` + `handleRequestType` -> 处理逻辑并生成数据\

#### 3、返回数据
 -> `LKS_ConnectionManager` -> `respond:requestType:tag:` -> `_sendData`\
 -> `Lookin_PTChannel` -> `sendFrameOfType` -> 相关数据组装 -> `dispatch_io_write`


#### 4、请求页面层次
 `LookinRequestTypeHierarchy` -> `[LookinHierarchyInfo staticInfoWithLookinVersion:clientVersion]` -> `displayItems: [LookinDisplayItem]` -> `UIApplication.sharedApplication.windows` -> `layer` + `sublayers`


查看结构：`LookinRequestTypeHierarchy`
 ```
 LookinDisplayItem
├── viewObject<LookinObject>
├── layerObject<LookinObject>
│   ├── memoryAddress<String>
│   └── classChainList<[String]>
│   └── completedSelfClassName<String>
└── subitems<[LookinDisplayItem]>
 ```

#### 5、查看UI元素UI内容截图
通过oid查找到layer，通过layer绘制图片

`LookinRequestTypeHierarchyDetails` -> `LKS_HierarchyDetailsHandler.startWithPackages` -> `_dequeueAndHandlePackage`\
-> `[NSObject lks_objectWithOid:task.oid]` -> `layer` -> `lks_soloScreenshotWithLowQuality` -> `groupScreenshot`+`soloScreenshot`-> `LookinDisplayItemDetail`


#### 6、修改属性
6.1 执行修改指令
`LookinRequestTypeInbuiltAttrModification` -> 
`LKS_InbuiltAttrModificationHandler` -> `handleModification` -> 
`LookinRequestTypeAttrModificationPatch` -> Runtime的方法转发 -> `NSMethodSignature` + `NSInvocation`

6.2 修改后的截图
`LookinRequestTypeAttrModificationPatch` -> `LKS_InbuiltAttrModificationHandler` -> `handlePatchWithTasks` \
-> `[NSObject lks_objectWithOid:task.oid]` -> `layer` -> `lks_soloScreenshotWithLowQuality` -> `groupScreenshot`+`soloScreenshot`-> `LookinDisplayItemDetail`

----

### 使用Xcode的debug模式下动态库注入实现sever
使用断点调试的功能，在Xcode中配置后`UIApplicationMain`运行时，使用`LLDB`执行加载命令加载动态库, 这样跟直接集成代码和动态加载一样都能实现创建一个Server
```
expr (Class)NSClassFromString(@"Lookin") == nil ? (void *)dlopen("/Applications/Lookin.app/Contents/Resources/LookinServerFramework/LookinServer.framework/LookinServer", 0x2) : ((void*)0)
```
![image.png](/assets/img/lookin/lookin6.png)

----
使用`optool`或`insert_dylib`对ipa进行注入动态库安装到真机