---
title: emoji非系统字体展示异常问题
author: 独孤流
date: 2024-01-11 01:04:00 +0800
categories: [other, 其他]
tags: [emoji]     # TAG names should always be lowercase
---

参考：
- https://developer.android.com/develop/ui/views/text-and-emoji/emoji-compat?hl=zh-cn
- [最新emoji表情列表](https://www.unicode.org/emoji/charts/full-emoji-list.html)\
emoji 12： https://unicode.org/emoji/charts-12.0/full-emoji-list.html\
https://www.webfx.com/tools/emoji-cheat-sheet/\
https://apps.timwhitlock.info/emoji/tables/unicode\

>前言
之前维护的一个APP，为了能和安卓、pc互通，有多个表情结合，
1、标准的系统emoji的表情集，本质上是标准的一系列emoji的Unicode列表
2、自定义的表情集，本质上是打包到APP里的一系列图片，这个的问题是在于打包写死的，不方便动态添加，这种功能应该一开始就做成接口下发比较灵活
3、用户自己收藏的表情，本质上收藏的是一个png\jpg\gif等图片的url地址
在几个月前产品提了一个需求，针对不同国家使用不同的字体，而且使用的都是非系统字体，结果这几天反馈

问题：
一些标准系统emoji在pc和安卓都正常，就在iOS上展示的没有底色，一开始没有仔细研究，还以为是需要富文本才可以展示完整的emoji,如🈷️🈶🈚️🈸🈺💮🉐㊗️🈴🈵🈹🈲🆘

解决方案：
但今天一个同事研究后确定是是因为我们没有使用系统字体导致，需要记录下，解决方案目前来说就是需要展示系统emoji的地方需要还原成系统字体即可

效果如下：
![image](/assets/img/other/emojifont1.png)
![image](/assets/img/other/emojifont2.png)
![image](/assets/img/other/emojifont3.png)