---
title: 生成安装ipa的自建安装服务端的plist
author: 独孤流
date: 2024-10-03 01:04:00 +0800
categories: [git_shell_resign, shell]
tags: [plist]     # TAG names should always be lowercase
---

> ### 前言
> 在开发好打包给测试时，一般都是使用自建服务器分发ipa，这个过程中就需要生成一个用于安装的plist，完整的安装地址`itms-services://?action=download-manifest&url=https://xxx.xx.xx/xxx/xxx.plist`

生成`xxx.plist`的完整demo：`testInstallPlist.sh`

```
#!/bin/sh
# 不包含脚本自身

plist_path=$1
icon57Path=$2
icon512Path=$3
ipaPath=$4
appBundleId=$5
appVersion=$6
appName=$7
appDesc=$8
# https://blog.csdn.net/WangErice/article/details/102599270
# https://juejin.cn/post/6844903716395417614
# 1.初始化一个plist文件
touch $plist_path
# 2.指定文件格式和编码格式
echo "<?xml version="1.0" encoding="UTF-8"?><plist version="1.0"><dict></dict></plist>" > $plist_path
/usr/libexec/PlistBuddy -c "Add :items array" $plist_path
/usr/libexec/PlistBuddy -c "Add :items: dict" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets array" $plist_path

/usr/libexec/PlistBuddy -c "Add :items:0:assets:0 dict" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets:0:kind string software-package" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets:0:url string $ipaPath" $plist_path

/usr/libexec/PlistBuddy -c "Add :items:0:assets:1 dict" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets:1:kind string display-image" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets:1:url string $icon57Path" $plist_path

/usr/libexec/PlistBuddy -c "Add :items:0:assets:2 dict" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets:2:kind string full-size-image" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:assets:2:url string $icon512Path" $plist_path

/usr/libexec/PlistBuddy -c "Add :items:0:metadata dict" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:bundle-identifier string $appBundleId" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:bundle-version string $appVersion" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:kind string software" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:subtitle string $appDesc" $plist_path
/usr/libexec/PlistBuddy -c "Add :items:0:metadata:title string $appName" $plist_path

# sh /Users/xxx/Downloads/testIpa/testInstallPlist.sh /Users/xxx/Downloads/testIpa/test.plist https://xx/xx/xx.png https://xx/xx/yy.png https://xx/xx/zz.ipa aa.bb.com 1.0.5 美好 美好一直存在
```