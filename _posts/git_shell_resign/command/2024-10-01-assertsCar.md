---
title: 解析Assets.car
author: 独孤流
date: 2024-10-01 02:04:00 +0800
categories: [git_shell_resign, shell]
tags: [Asserts.car, ipa]     # TAG names should always be lowercase
---

参考：
- [Analysing Assets.car file in iOS](https://stackoverflow.com/questions/22630418/analysing-assets-car-file-in-ios)
- [acextract]([acextract](https://github.com/bartoszj/acextract))
- [ThemeEngine](https://github.com/alexzielenski/ThemeEngine)
- [iOS Asset Extractor](https://github.com/Marxon13/iOS-Asset-Extractor)
- [Asset Catalog Tinkerer](https://github.com/insidegui/AssetCatalogTinkerer)
- [cartool](https://github.com/steventroughtonsmith/cartool)
- [QLCARFiles -- 亲测可用](https://github.com/Timac/QLCARFiles)
  

> ### 前言
> 在做解析ipa自动生成安装链接时，需要读取ipa的logo，特需要解析`Assets.car`


#### 方式一：使用系统提供的工具
```
xcrun --sdk iphoneos assetutil --info Assets.car
```

#### 方式二、使用三方工具，推荐 [QLCARFiles -- 亲测可用](https://github.com/Timac/QLCARFiles)
1、下载源代码 [https://github.com/Timac/QLCARFiles](https://github.com/Timac/QLCARFiles)
2、运行项目
3、`Product` -> `Show Build Folder in Finder` -> `/Users/xxx/Library/Developer/Xcode/DerivedData/QLCARFiles-xxxxxx/Build/Products/Debug/carDump`

![image](/assets/img/terminal/carDump0.png)
![image](/assets/img/terminal/carDump1.png)
![image](/assets/img/terminal/carDump2.png)
![image](/assets/img/terminal/carDump3.png)

完整demo：`testAssertsCar.sh`
```
#!/bin/sh

# 读取当前脚本的路径
script_path=$0
script_dir_path=$(cd "$(dirname "$0")";pwd)

# Asserts.car路径
assertsPath=$1
if [ ! -f "$assertsPath" ]; then
    echo "未找到asserts.car $assertsPath"
    exit 2
fi

tempAssertsDirPath="${script_dir_path}/tempIpaUnzipDir"
echo "tempAssertsDirPath: $tempAssertsDirPath"
if [ -d "$tempAssertsDirPath" ]; then
    echo "删除临时asserts解包目录 rm ${tempAssertsDirPath}"
    rm -rf "${tempAssertsDirPath}"
    echo "asserts解包文件夹删除完成后重新创建文件夹"
    mkdir $tempAssertsDirPath
else
    echo "asserts解包文件夹不存在，创建文件夹"
    mkdir $tempAssertsDirPath
fi


# 将AppIcno*png复制到其他文件夹
iconDir="${script_dir_path}/appicons"
if [ -d "$iconDir" ]; then
    echo "删除临时iconDir解包目录 rm ${iconDir}"
    rm -rf "${iconDir}"
    echo "iconDir解包文件夹删除完成后重新创建文件夹"
    mkdir $iconDir
else
    echo "asserts解包文件夹不存在，创建文件夹"
    mkdir $iconDir
fi


if [ -f $assertsPath ];then
    # 正式解压
    carDumpPath="${script_dir_path}/carDump"
    if [ -f $carDumpPath ];then
        $carDumpPath $assertsPath $tempAssertsDirPath
        # 查找AppIcon的图片
        # find $tempAssertsDirPath -name "AppIcon*.png"
        find $tempAssertsDirPath -name "AppIcon*.png" -print0 | xargs -0 -J % cp % "$iconDir"
    fi
else
    echo "$assertsPath 不存在"
fi

lastFindIcon="${iconDir}/findIcon.png"
if [ "$(ls -A $iconDir)" ]; then
    echo "$iconDir is not Empty"
    # 查找最大的文件路径
    findIconPath=$(du -s ${iconDir}/* | sort -nr | head -1)
    echo "findIconPathWidthSize: $findIconPath"
    # 去掉文件路径里的大小，恢复为标准的路径
    findIconPath="/Users/${findIconPath#*Users/}"
    echo "findIconPath: $findIconPath"
    cp $findIconPath "${lastFindIcon}"
fi

# sh /Users/xxx/Downloads/testIpa/testAssertsCar.sh /Users/xxx/Downloads/testIpa/Assets.car
```
