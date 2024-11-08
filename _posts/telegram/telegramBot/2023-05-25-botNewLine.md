---
title: TelegramBot发送消息内容换行支持
author: 独孤流
date: 2023-05-25 08:04:00 +0800
categories: [Telegram, TelegramBot]
tags: [Telegram_bot]     # TAG names should always be lowercase
---

> ### 前言
> 在发送消息时，如果文字太长会很乱，一些信息主动换行逻辑比较才比较清晰，但是一直没找到方法让文字换行

让文字换行，经过研究有2种方式

#### 方式一，使用URL编码换行符换行符`%0A`
```
bot_token="xxxxx"
chat_id=xxxx
# 由于这里是使用curl发送的，所以支持URL编码的%0A
msg="name:tom%0Aage:100"
curl -s -X POST \
"https://api.telegram.org/bot$bot_token/sendMessage?chat_id=$chat_id" -d text="$msg"
```
![image](/assets/img/telegramBot/bot_line1.png)

#### 方式二，直接在文本里换行
```
bot_token="xxxx"
chat_id=xxxx
# 换行符本质是\n,但如果直接输入\n,其他输入的是\\n这个
msg="addr:wod
Hobby:hiking"
curl -s -X POST \
"https://api.telegram.org/bot$bot_token/sendMessage?chat_id=$chat_id" -d text="$msg"
```
![image](/assets/img/telegramBot/bot_line2.png)