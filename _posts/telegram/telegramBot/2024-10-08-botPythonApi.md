---
title: python-telegram-bot接收指令一
author: 独孤流
date: 2024-10-08 08:04:00 +0800
categories: [Telegram, TelegramBot]
tags: [Telegram_bot]     # TAG names should always be lowercase
---

参考：
- [python-telegram-bot](https://python-telegram-bot.org/)
- [docs.python-telegram-bot.org](https://docs.python-telegram-bot.org/en/v21.6/)
- [https://core.telegram.org/api](https://core.telegram.org/api)
- [https://github.com/python-telegram-bot/python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)
- [在Python应用中Telegram 机器人搭建消息提醒](https://foofish.net/telegram-bot-send-message.html)
- [Python Telegram Bot](https://piaohua.github.io/post/python/20231230-telegram-bot/)
- [Telegram Bot 简明教程 II - 收指令与指令键盘](https://clox.nu/blog/brief-tutorial-on-telegram-bot-ii/)
- [Telegram Bot 简明教程 I - 注册与发消息](https://clox.nu/blog/brief-tutorial-on-telegram-bot-i/)


### 一、安装`python-telegram-bot`
```
# python 2.x环境的
pip install python-telegram-bot --upgrade
# python 3.x环境的
pip3 install python-telegram-bot --upgrade
```

----
#### 二、简单的接收指令及解析消息的例子`testBot1.py`
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler

# 处理 /start 命令
async def start(update: Update, context) -> None:
    print(update)
    # print("======message======")
    # print(update.message)
    print("=======from_user=====")
    print(update.message.from_user)
    print("======chat======")
    print(update.message.chat)
    print("======text======")
    print(update.message.text)
    # 回复消息
    await update.message.reply_text('你好！我是你的 Telegram 机器人！=>' + update.message.text)

if __name__ == '__main__':
    application = ApplicationBuilder().token('your token').build()

    # 添加 /start 命令处理器
    application.add_handler(CommandHandler('start', start))

    # 运行机器人
    application.run_polling()

# python3 /xxx/xxx/testBot1.py
# ctrl + c 停止运行
```
![image](/assets/img/telegramBot/botApi1.png)

![image](/assets/img/telegramBot/botApi2.png)

----

#### 三、展示键盘选择的例子`testBot2.py`
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler

# 处理 /start 命令
async def start(update: Update, context) -> None:
    # print("=====start=update======")
    # print(update)
    # print("======message======")
    # print(update.message)
    print("=======from_user=====")
    print(update.message.from_user)
    print("======chat======")
    print(update.message.chat)
    print("======text======")
    print(update.message.text)
    keyboard = [[InlineKeyboardButton("选项 1", callback_data='1'),
                 InlineKeyboardButton("选项 2", callback_data='2')]]

    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text('请选择一个选项：', reply_markup=reply_markup)

# 处理按钮回调
async def button(update: Update, context) -> None:
    # print("=====button=update======")
    # print(update)
    print("=======callback_query=====")
    print(update.callback_query)
    print("=======data=====")
    print(update.callback_query.data)
    print("=======message=====")
    print(update.callback_query.message)
    print("=======from_user=====")
    print(update.callback_query.from_user)
    query = update.callback_query
    await query.answer()
    # 编辑回复
    await query.edit_message_text(text=f"你选择了选项 {query.data}")

if __name__ == '__main__':
    application = ApplicationBuilder().token('your token').build()

     # 添加处理命令和按钮的处理器
    application.add_handler(CommandHandler('start', start))
    application.add_handler(CallbackQueryHandler(button))

    # 运行机器人
    application.run_polling()

# python3 /xxx/xxx/testBot2.py
# ctrl + c 停止运行
```

----
#### 四