---
title: python-telegram-bot交互二
author: 独孤流
date: 2024-10-09 08:04:00 +0800
categories: [Telegram, TelegramBot]
tags: [Telegram_bot]     # TAG names should always be lowercase
---

参考：
- [python-telegram-bot](https://python-telegram-bot.org/)
- [docs.python-telegram-bot.org](https://docs.python-telegram-bot.org/en/v21.6/)
- [https://core.telegram.org/api](https://core.telegram.org/api)
- [Introduction-to-the-API](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Introduction-to-the-API)
- [Your-first-Bot](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Extensions---Your-first-Bot)

### API说明：\
1、`telegram.Bot("TOKEN")`: 初始化Bot

2、` bot.get_me()`: 获取机器人的基本信息

3、`bot.get_updates()`: 获取更新的信息

4、`bot.send_message(text='Hi John!', chat_id=1234567890)`: 给指定id的人活群发消息

完整demo：`testBot3.py`
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import asyncio
import telegram


async def main():
    bot = telegram.Bot("xxxx")
    async with bot:
        # 获取机器人的信息
        botMe = await bot.get_me()
        print(botMe)
        # 获取最新的消息
        updates = (await bot.get_updates())
        print(updates)
        # 给某个人或某个人发消息
        await bot.send_message(text='hello world', chat_id=xxxx)


if __name__ == '__main__':
    asyncio.run(main())

# python3 /xxx/xxx/testBot3.py
# ctrl + c 停止运行
```

----

### `telegram.ext`
> `telegram.ext`模块建立在纯 API 实现之上。它提供了一个易于使用的接口，并减轻了程序员的工作量
> 它由几个类组成，但最重要的是`telegram.ext.Application`
>
> `Application` 类负责从 `update_queue` 中获取更新，`Update`r 类会不断从 `Telegram` 获取新更新并将其添加到此队列中。如果您使用 `ApplicationBuilder` 创建 `Application` 对象，它将自动为您创建一个 `Updater` 并将它们与 `asyncio.Queue` 链接在一起。然后，您可以在 `Application` 中注册不同类型的处理程序，它将根据您注册的处理程序对 `Updater` 获取的更新进行排序，并将它们传递给您定义的回调函数。
>
> 每个处理程序都是 `telegram.ext.BaseHandler`类的任何子类的实例。该库为几乎所有用例提供了处理程序类，但如果您需要非常具体的东西，您也可以自己子类化 `Handler`。