---
title: python异步执行shell脚本
author: 独孤流
date: 2024-10-03 01:04:00 +0800
categories: [git_shell_resign, python]
tags: [python]     # TAG names should always be lowercase
---

```
#!/usr/bin/env python
import asyncio

# 异步执行命令，并实时获取输出
async def run_command_realtime(command: str):
    process = await asyncio.create_subprocess_shell(
        command,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE
    )

    # 创建两个异步任务，用于分别读取 stdout 和 stderr 的输出
    await asyncio.gather(
        stream_output(process.stdout, "输出"),
        stream_output(process.stderr, "错误"),
    )

    # 等待进程结束，并获取退出状态码
    returncode = await process.wait()
    if returncode == 0:
        print("命令执行完成 ✅")
    else:
        print(f"命令执行失败 ❌，退出码: {returncode}")

# 实时读取流（stdout/stderr）并逐行发送到 Telegram
async def stream_output(stream, stream_name: str):
   while True:
        chunk = await stream.read(1024)  # 每次读取 1024 字节
        if not chunk:  # 如果没有更多数据，跳出循环
            break
        # 解码时忽略解码错误
        text = chunk.decode('utf-8', errors='ignore').strip()
        if text:
            print(f"[{stream_name}] `{text}`")

def main() -> None:
   print("hello")
   # 开始异步执行脚本
   # 异步执行脚本，不阻塞流程
   commandStr = "sh /xxx/xx/xxx.sh abc x.y.c -234234"
   asyncio.create_task(run_command_realtime(commandStr))
   # 执行其他逻辑
   print("world")


if __name__ == "__main__":
    main()

# python3 /xxx/xxx/xxx.py
```

----
异步执行的测试脚本内容`async.sh`
```
#!/bin/bash
export LC_ALL=en_US.UTF-8
# 不包含脚本自身
echo "输入的命令：$0 $@"
echo "参数个数：$#+1"
# 便利输入的参数
idx=0
echo "参数位置:$idx 参数内容: $0"
for i in $@; do
    idx=$(($idx+1))
    echo "参数位置:$idx 参数内容: $i"
done

script_path=$0
script_dir_path=$(cd "$(dirname "$0")";pwd)
SCHEME=$1
newBundleId=$2
chat_id=$3
time=$(date "+%Y-%m-%d %H:%M:%S")
echo "sleep开始 $time"
sleep 10

bot_token="xxxxx"
# 发送文件
resign_ipa_path="/xxx/xxx/xx.txt"
desc="bundleId: ${newBundleId} SCHEME: ${SCHEME}"
curl -s -X POST "https://api.telegram.org/bot${bot_token}/sendDocument" -H "Content-Type:multipart/form-data" -F chat_id=$chat_id  -F document=@"${resign_ipa_path}" -F caption="$desc"

time2=$(date "+%Y-%m-%d %H:%M:%S")
echo "sleep结束 $time2"
```