---
title: TelegramBot创建
author: 独孤流
date: 2023-05-24 08:04:00 +0800
categories: [Telegram, TelegramBot]
tags: [Telegram_bot]     # TAG names should always be lowercase
---

### 一、使用Telegram机器人发送消息
2.1 生成并创建Telegram机器人
参考：
- [Telegram 机器人的申请和设置图文教程](https://www.dengnz.com/2020/11/23/telegram-%E6%9C%BA%E5%99%A8%E4%BA%BA%E7%9A%84%E7%94%B3%E8%AF%B7%E5%92%8C%E8%AE%BE%E7%BD%AE%E5%9B%BE%E6%96%87%E6%95%99%E7%A8%8B/)
- [Telegram：新手指南、使用教程及频道推荐（持续更新中）](https://pipuwong.com/telegram)
- [Telegram Bot 简明教程 I - 注册与发消息](https://clox.nu/blog/brief-tutorial-on-telegram-bot-i/)

##### 1.1 首先在Telegram 里搜索 `https://telegram.me/BotFather`
![image](/assets/img/telegramBot/bot1.png)

##### 1.2 输入 `/newbot`

输入机器人名字，必须要以`bot`结尾
> Good. Now let's choose a username for your bot. It must end in `bot`. Like this, for example: TetrisBot or tetris_bot.

名字输入好后，保存好Token信息
![image](/assets/img/telegramBot/bot2.png)

##### 1.3 输入 `/mybots` 可以编辑相关信息


----
### 二、获取机器人收到的最新消息

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
![image](/assets/img/telegramBot/bot3.png)
Telegram机器人获取个人及群组ID
----
### 三、使用机器人发送消息
2.2 使用机器人发送通知消息
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


# 编辑消息
curl -X POST "https://api.telegram.org/bot<Your_Bot_API_Token>/editMessageText" \
     -d "chat_id=<chat_id>" \
     -d "message_id=<message_id>" \
     -d "text=<new_message_text>"

# 删除消息
curl -X POST "https://api.telegram.org/bot<Your_Bot_API_Token>/deleteMessage" \
     -d "chat_id=<chat_id>" \
     -d "message_id=<message_id>"

```