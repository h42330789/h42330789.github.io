---
title: 解析Assets.car
author: 独孤流
date: 2024-10-01 02:04:00 +0800
categories: [other, 其他]
tags: [Assets]     # TAG names should always be lowercase
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