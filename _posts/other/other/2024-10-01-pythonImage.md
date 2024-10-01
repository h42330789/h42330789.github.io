---
title: Python生成各种图片的方法
author: 独孤流
date: 2024-10-01 01:04:00 +0800
categories: [other, 其他]
tags: [python]     # TAG names should always be lowercase
---

参考：
- [PIL中的paste方法拼接透明背景照片](https://blog.csdn.net/tscaxx/article/details/126361568)
- [Python进行图片缩放](https://blog.csdn.net/u010889616/article/details/79437403)
- [python 根据文字生成图片_mob649e81643021 的技术博客_51CTO 博客](https://blog.51cto.com/u_16175499/6851032)
- [QLCARFiles](https://github.com/Timac/QLCARFiles)


> ### 前言
> 在解压ipa自动生成安装链接过程中，如果发现ipa里没有logo时，自动使用名字生成一个图片

### 一、生成一个空白的图片
```
# 文字设置图片
def createImg(text, fontSize, width, height, outPath):
    #画布颜色,背景颜色为白色
    image = Image.new('RGB', (width, height), (255, 255, 255))
    draw = ImageDraw.Draw(image)
    # mac里有这个字体，但是中文会乱码，如果需要中文正常，设置个指定支持的中文字体即可
    # font = ImageFont.truetype("/xxx/xxx/xxx.ttf", fontSize)
    font = ImageFont.truetype('Arial.ttf', fontSize)
    # 获取文字使用该字体占据的控件
    x, y, text_width, text_height = font.getbbox(text)
    # 将文字在画布的中间展示
    draw.text(((width-text_width)/2, (height-text_height)/2), text, fill=(0, 0, 0), font=font)
    # 保存图片
    image.save(outPath)
```

### 二、压缩图片
```
# 图片压缩
def resizeImage(inPath, width, height, outPath):
    # 原始图片
    img = Image.open(inPath)
    # 将图片设置成新的宽高
    out = img.resize((width,height))
    # 保存图片
    out.save(outPath)
```

### 三、在已有图片上添加图片，用于图片叠加、增加角标等
```
# 图片上添加图片
def imageAddImage(inPath, addPath, outPath):
    background_image = Image.open(inPath).convert("RGBA")
    # 打开要添加的图片
    add_image = Image.open(addPath).convert("RGBA")
    result_image = Image.new("RGBA", background_image.size)
    # 将背景图片粘贴到新的图片对象上
    result_image.paste(background_image, (0, 0))
    # 将要添加的图片粘贴到新的图片对象上
    # 放到右上角
    x = background_image.size[0] - add_image.size[0]
    y = 0
    # 将角标图片放到右上角，这里设置两个参数add_image才能展示出透明效果
    result_image.paste(add_image, (x, y), add_image)
    # 保存结果图片
    result_image.save(outPath)
```
----
完整demo:`makeImage.py`
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import sys
import os
from PIL import Image, ImageFont, ImageDraw
 
# 文字设置图片
def createImg(text, fontSize, width, height, outPath):
    #画布颜色,背景颜色为白色
    image = Image.new('RGB', (width, height), (255, 255, 255))
    draw = ImageDraw.Draw(image)
    # mac里有这个字体，但是中文会乱码，如果需要中文正常，设置个指定支持的中文字体即可
    # base_dir = os.path.dirname(os.path.abspath(__file__))
    # font = ImageFont.truetype(base_dir + "/xxxx.ttf", fontSize)

    font = ImageFont.truetype('Arial.ttf', fontSize)
    # 获取文字使用该字体占据的控件
    x, y, text_width, text_height = font.getbbox(text)
    # 将文字在画布的中间展示
    draw.text(((width-text_width)/2, (height-text_height)/2), text, fill=(0, 0, 0), font=font)
    # 保存图片
    image.save(outPath)

# 图片压缩
def resizeImage(inPath, width, height, outPath):
    img = Image.open(inPath)
    print(img.size)
    out = img.resize((width,height))
    out.save(outPath)
 
# 图片上添加图片
def imageAddImage(inPath, addPath, outPath):
    background_image = Image.open(inPath).convert("RGBA")
    # 打开要添加的图片
    add_image = Image.open(addPath).convert("RGBA")
    result_image = Image.new("RGBA", background_image.size)
    # 将背景图片粘贴到新的图片对象上
    result_image.paste(background_image, (0, 0))
    # 将要添加的图片粘贴到新的图片对象上
    # 放到右下角
    x = background_image.size[0] - add_image.size[0]
    y = 0 - 20
    result_image.paste(add_image, (x, y), add_image)
    # 保存结果图片
    result_image.save(outPath)

print(sys.argv)
scriptPath = sys.argv[0]
commandType = sys.argv[1]
if "createImg" in commandType:
    text = sys.argv[2]
    width = sys.argv[3]
    fontSize = sys.argv[4]
    outPath = sys.argv[5]
    wh = int(width)
    createImg(text, int(fontSize), wh, wh, outPath)
elif "resizeImage" in commandType:
    inPath = sys.argv[2]
    width = sys.argv[3]
    outPath = sys.argv[4]
    wh = int(width)
    resizeImage(inPath, wh, wh, outPath)
elif "imageAddImage" in commandType:
    inPath = sys.argv[2]
    addPath = sys.argv[3]
    outPath = sys.argv[4]
    imageAddImage(inPath, addPath, outPath)

# 生成一个512x512，字体大小为60， 图片上展示文字内容为hello的图片
# python3 /Users/xxx/xxxx/makeImage.py createImg "hello" 512 60 /Users/xxx/xxx/xxx.png
# aa.png上添加bb.png,保存为cc.png
# python3 /Users/xxx/xxxx/makeImage.py imageAddImage /xxx/xxx/aa.png xxx/xxx/bbb.png /xxx/xxx/ccc.png
# 将aa.png的图片的大小压缩为57x57大小的图片
# python3 /Users/xxx/xxxx/makeImage.py resizeImage 57 /xxx/xxx/aa.png 57 /xxx/xxx/bb.png

```
如果遇到 `ModuleNotFoundError: No module named xxx,`导入即可
```
# python
# ModuleNotFoundError: No module named 'PIL'
# pip install pillow
# python3
pip3 install pillow
```