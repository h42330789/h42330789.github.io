---
title: Telegram机器人一
author: 独孤流
date: 2024-01-29 01:04:00 +0800
categories: [Telegram]
tags: [Telegram, Bot]     # TAG names should always be lowercase
---


1、使用Telegram机器人发送消息
1.1 生成并创建Telegram机器人
参考：Telegram 机器人的申请和设置图文教程

首先在Telegram 里搜索 `@botfather`
输入 `/newbot`
输入 `/mybots` 可以编辑相关信息

打开浏览器，输入以下链接：`https://api.telegram.org/bot<YourBOTToken>/getUpdates`
```
# https://api.telegram.org/bot<YourBOTToken>/getUpdates
# 假如token为bot6072345345:AAdfgsdfgsdfg345Bl97_-io
# 完整地址如下
https://api.telegram.org/bot6072345345:AAdfgsdfgsdfg345Bl97_-io/getUpdates
# 通过返回的数据里，username、title等关键字就能知道对应的人，id则是对应个人或群的id，个人id是正数，群id是负数
{"ok":true,"result":[{"update_id":6346xxx9,
"message":{"message_id":21xx,"from":{"id":5970xxxx,"is_bot":false,"first_name":"axxx","username":"axxxx"},"chat":{"id":-73xxxx,"title":"xxxxx",
 "type":"group","all_members_are_administrators":true}}}......]}
```
Telegram机器人获取个人及群组ID

1.2 使用机器人发送通知消息
api文档：https://core.telegram.org/bots/api
```
# 发送图片
bot_token="xxxx"
imgPath="xxx/xxx.png"
group_id="xxxx"
notify_msg="hello_everyone"
# 发送图片
curl -s -X POST "https://api.telegram.org/bot$bot_token/sendPhoto" -F chat_id=$group_id -F photo="@$imgPath" -F caption=$notify_msg

# 发送文字信息
curl -s -X POST \
"https://api.telegram.org/bot$bot_token/sendMessage?chat_id=$group_id" -d parse_mode="HTML" \
-d text="$notify_msg"
```
参考：https://pipuwong.com/telegram