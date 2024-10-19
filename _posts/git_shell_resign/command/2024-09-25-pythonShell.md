---
title: python读取执行shell后的内容
author: 独孤流
date: 2024-09-25 01:04:00 +0800
categories: [git_shell_resign, python]
tags: [terminal, python]     # TAG names should always be lowercase
---

参考：
- [python中得到shell命令输出的方法](https://blog.csdn.net/wanglei_storage/article/details/54615952)

读取执行脚本命令的输出
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import os

# 读取执行命令后的输出内容
outputFile = os.popen('pwd') # return file
outputStr = outputFile.read()
outputFile.close()
print("outputStr: " + outputStr)



# 读取执行命令的结果
status = os.system("pwd")
// print str\int混排的方式
print("status:", status)
print("status:" + str(status))
print(f"status: {status}")
if status == 256:
    print("errorr")
else:
    print("ok")

```