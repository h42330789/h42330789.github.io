---
title: applescript模拟人操作xcode打包
author: 独孤流
date: 2024-10-03 016:04:00 +0800
categories: [git_shell_resign, shell]
tags: [applescript]     # TAG names should always be lowercase
---

参考：
- [AppleScript 入门：探索 macOS 自动化](https://sspai.com/post/46912)
- [AppleScript脚本快速入门](https://juejin.cn/post/7055599089081122829)

> ### 前言
> 听人说他们iOS开发者账号莫名其妙被封了好几个，封了后又要重新花钱购买，操作麻烦还费钱
> 偶然发现虽然账号被封了，但是证书和描述文件还可以使用`Xcode`手动操作打包，但是使用`xcodebuild`就会报错，于是研究了下使用`AppleScript`来模拟手动操作打包

具体代码如下：`test.scpt`, 先使用mac自带的`脚步编辑器`软件创建一个applescript脚步，然后将如下代码copy进入
```
on run {codePath, scheme}
	openXcode(codePath)
	delay 5 -- 根据需要调整加载时间
	changeScheme(scheme)
end run


on openXcode(path)
	tell application "Xcode"
		activate
		open path
	end tell
end openXcode

on changeScheme(scheme)
	tell application "System Events"
		tell process "Xcode"
			-- 打开 Scheme 菜单
			click menu item "Scheme" of menu 1 of menu bar item "Product" of menu bar 1
			delay 3
			
			-- 模拟选择指定的 Scheme
			click menu item scheme of menu 1 of menu item "Scheme" of menu 1 of menu bar item "Product" of menu bar 1 -- 替换为 Scheme 名称
			delay 3
		end tell
	end tell
	
	-- 执行 Archive（归档）
	tell application "System Events"
		tell process "Xcode"
			-- 点击 Product > Archive 菜单项
			click menu item "Archive" of menu 1 of menu bar item "Product" of menu bar 1
		end tell
	end tell
end changeScheme
```

使用shell脚本调用`applescript`,demo: `test.sh`
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
codePath="/xxx/xxxx/xxx.xcworkspace"
scheme=$1
# 执行applescript
osascript /xxx/xxx/test.scpt "$codePath" "$scheme"

# sh /xxxx/xxx/xxx.sh "xxxx"

```