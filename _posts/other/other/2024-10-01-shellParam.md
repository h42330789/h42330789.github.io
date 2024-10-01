---
title: Bash/Shell/Python读取参数及获取基本路径
author: 独孤流
date: 2024-10-01 04:04:00 +0800
categories: [other, 其他]
tags: [Shell]     # TAG names should always be lowercase
---

> ### 前言
> 经常写各种各种脚本时获取一些基本参数时经常忘记写法，特整理一下


#### 一、shell/bash获取参数相关`test.sh`

参考：
- [sh获取所有传递参数_如何在Linux中给shell脚本传参数，位置参数变量简单应用案例](https://blog.csdn.net/weixin_39884832/article/details/113039246)
- [shell脚本获取并打印所有参数](https://blog.csdn.net/Sheleon1995/article/details/126763677)

```
#!/bin/sh
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
# 0代表命令本身，1-9代表第1-9个参数 $0, $1, ... $9
# 10以上的参数要用大括号，如：${17}, ${10}, ...
# 读取当前脚本的路径
script_path=$0
script_dir_path=$(cd "$(dirname "$0")";pwd)
# 获取参数，这个位置是会包含脚本自身，$1就是$@的第0个参数
ipa_in_path=$1
# 读取最后路径的最后/后面的部分
ipa_file_name="$(basename "$ipa_in_path")"
# 读取路径最后/后面的部分，同时替换掉指定的最后部分
ipa_name="$(basename "$ipa_in_path" ".ipa")"

echo "script_path: $script_path"
echo "script_dir_path: $script_dir_path"
echo "ipa_in_path: $ipa_in_path"
echo "ipa_file_name: $ipa_file_name"
echo "ipa_name: $ipa_name"

# sh /Users/xxx/Downloads/testIpa/test.sh /Users/xxx/Desktop/xx/yy/aaa.ipa a b c 4 5 6 7 8 hello 你好 123.03

```
执行效果
```
输入的命令：/Users/xxx/Downloads/testIpa/test.sh /Users/xxx/Desktop/xx/yy/aaa.ipa a b c 4 5 6 7 8 hello 你好 123.03
参数个数：12+1
参数位置:0 参数内容: /Users/xxx/Downloads/testIpa/test.sh
参数位置:1 参数内容: /Users/xxx/Desktop/xx/yy/aaa.ipa
参数位置:2 参数内容: a
参数位置:3 参数内容: b
参数位置:4 参数内容: c
参数位置:5 参数内容: 4
参数位置:6 参数内容: 5
参数位置:7 参数内容: 6
参数位置:8 参数内容: 7
参数位置:9 参数内容: 8
参数位置:10 参数内容: hello
参数位置:11 参数内容: 你好
参数位置:12 参数内容: 123.03
script_path: /Users/xxx/Downloads/testIpa/test.sh
script_dir_path: /Users/xxx/Downloads/testIpa
ipa_in_path: /Users/xxx/Desktop/xx/yy/aaa.ipa
ipa_file_name: aaa.ipa
ipa_name: aaa
```

----

### Python获取参数`test.py`
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import os
import sys

# 不包含脚本自身
print (sys.argv)
argvLen = len(sys.argv)
print("参数个数：" + str(argvLen))
idx = 0
for arg in sys.argv:
    print("参数位置: " + str(idx) +  "参数内容: " + str(arg))
    idx = idx + 1

current_file_path = __file__
print("current_file_path: " + current_file_path)
script_path = sys.argv[0]
script_dir_path = os.path.dirname(os.path.abspath(script_path))
# 获取参数，这个位置是会包含脚本自身
ipa_in_path = sys.argv[1]
# 读取最后路径的最后/后面的部分
(prePath, ipa_file_name) = os.path.split(ipa_in_path)
# # 读取路径最后/后面的部分，同时替换掉指定的最后部分
(ipa_name, ext) = os.path.splitext(ipa_file_name)

print("script_path: " + script_path)
print("script_dir_path: " + script_dir_path)
print("ipa_in_path: " + ipa_in_path)
print("prePath: " + prePath)
print("ipa_file_name: " + ipa_file_name)
print("ipa_name: " + ipa_name)
print("ext: " + ext)

# python /Users/xxx/Downloads/testIpa/test.py /Users/xxx/Desktop/xx/yy/aaa.ipa a b c 4 5 6 7 8 hello 你好 123.03
```
执行效果如下：
```
['/Users/xxx/Downloads/testIpa/test.py', '/Users/xxx/Desktop/xx/yy/aaa.ipa', 'a', 'b', 'c', '4', '5', '6', '7', '8', 'hello', '\xe4\xbd\xa0\xe5\xa5\xbd', '123.03']
参数个数：13
参数位置: 0参数内容: /Users/xxx/Downloads/testIpa/test.py
参数位置: 1参数内容: /Users/xxx/Desktop/xx/yy/aaa.ipa
参数位置: 2参数内容: a
参数位置: 3参数内容: b
参数位置: 4参数内容: c
参数位置: 5参数内容: 4
参数位置: 6参数内容: 5
参数位置: 7参数内容: 6
参数位置: 8参数内容: 7
参数位置: 9参数内容: 8
参数位置: 10参数内容: hello
参数位置: 11参数内容: 你好
参数位置: 12参数内容: 123.03
current_file_path: /Users/xxx/Downloads/testIpa/test.py
script_path: /Users/xxx/Downloads/testIpa/test.py
script_dir_path: /Users/xxx/Downloads/testIpa
ipa_in_path: /Users/xxx/Desktop/xx/yy/aaa.ipa
prePath: /Users/xxx/Desktop/xx/yy
ipa_file_name: aaa.ipa
ipa_name: aaa
ext: .ipa
```