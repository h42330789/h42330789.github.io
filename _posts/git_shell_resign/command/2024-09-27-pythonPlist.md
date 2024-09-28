---
title: python读取Plist的内容
author: 独孤流
date: 2024-09-27 01:04:00 +0800
categories: [git_shell_resign, shell]
tags: [terminal, python, plist]     # TAG names should always be lowercase
---

参考：
- [iOS小技能：使用PlistBuddy/plutil进行增删改查plist文件](https://juejin.cn/post/7151969500424831012)
- [【python】str与json类型转换](https://blog.csdn.net/lluozh2015/article/details/75092877)
- [PlistBuddy基本使用方法](https://juejin.cn/post/6844903716395417614)

> ### 前言
> 为了项目打包动态化，需要在Jenkins大包时根据App类型、App环境动态配置scheme，防止测试环境的安装包与线上环境的安装包有一样的scheme，如果有一样的scheme就会导致其他App拉起的App可能未必是期望的App


#### 一、使用`plutil`读取全部的plist内容为json格式的数据
```
def readPlist2Json(plistPath):
    # iOS小技能：使用PlistBuddy/plutil进行增删改查plist文件
    # https://juejin.cn/post/7151969500424831012
    # 读取执行命令后的输出内容
    # plutil -p
    exeStr = "/usr/bin/plutil -p %s" % (plistPath)
    print("exeStr: " + exeStr)

    jsonStr = ""
    with os.popen(exeStr) as input_file: 
        linesList = input_file.readlines()
        totalCount = len(linesList)
        for i in range(0,totalCount):
            lineStr = linesList[i]
            if "=>" in lineStr:
                keyVList = lineStr.split('=>')
                keyStr = keyVList[0]
                valStr = keyVList[1]
                valStr = valStr.replace("\n", "")
                # 有key的行
                if "\" =>" in lineStr:
                    #key是String，属于dict的key
                    if " => [" in lineStr:
                        # value是一个array
                        jsonStr = jsonStr + keyStr + ":" + valStr + "\n"
                    elif " => {" in lineStr:
                        # value是一个dict
                        jsonStr = jsonStr + keyStr + ":" + valStr + "\n"
                    else:
                        # value是一个普通的值
                        jsonStr = jsonStr + keyStr + ":" + valStr + "," + "\n"
                else:
                    #属于Array的index
                    #key是String，属于dict的key
                    if " => [" in lineStr:
                        # value是一个array
                        jsonStr = jsonStr + valStr + "\n"
                    elif " => {" in lineStr:
                        # value是一个dict
                        jsonStr = jsonStr + valStr + "\n"
                    else:
                        # value是一个普通的值
                        jsonStr = jsonStr + valStr + "," + "\n"
            else:
                lineStr2 = lineStr.replace("\n", "")
                if i == totalCount - 1 :
                    jsonStr = jsonStr + lineStr2
                else:
                    if "]" in lineStr:
                        jsonStr = jsonStr + lineStr2 + "," + "\n"
                    elif "}" in lineStr:
                        jsonStr = jsonStr + lineStr2 + "," + "\n"
                    else:
                        jsonStr = jsonStr + lineStr2 + "\n"


    # 【python】str与json类型转换
    # https://blog.csdn.net/lluozh2015/article/details/75092877
    json1 = eval(jsonStr)
    # print("The type of json1 is: ", type(json1))
    return json1
```

#### 二、使用`PlistBuddy`修改或添加`Info.plist`里的scheme
```
def changeInfoPlistScheme(infoPath, schemeName, schemeUrl):
    # PlistBuddy基本使用方法
    # https://juejin.cn/post/6844903716395417614
    plistDict = readPlist2Json(plistPath)
    CFBundleURLTypes = plistDict["CFBundleURLTypes"]
    totalCount = len(CFBundleURLTypes)
    findSchemeIndex = -1
    for i in range(0,totalCount):
        schemeDict = CFBundleURLTypes[i]
        # print(schemeDict)
        CFBundleURLName = schemeDict["CFBundleURLName"]
        # 严格相等的判断
        # if CFBundleURLName == schemeName:
        #     findSchemeIndex = i
        #     break
        # 包含前缀的判断
        if schemeName in CFBundleURLName:
            findSchemeIndex = i
            break
    
    status = 0
    if findSchemeIndex != -1:
        changeSchemeUrlKey = ":CFBundleURLTypes:%s:CFBundleURLSchemes:0" % (findSchemeIndex)
        changeSchemeUrlExeStr = "/usr/libexec/PlistBuddy -c \"Set %s %s\" %s" % (changeSchemeUrlKey, schemeUrl, infoPath)
        print("changeSchemeUrlExeStr: " + changeSchemeUrlExeStr)
        status = os.system(changeSchemeUrlExeStr)
        # changeSchemeNameKey = ":CFBundleURLTypes:%s:CFBundleURLName" % (findSchemeIndex)
        # changeSchemeNameExeStr = "/usr/libexec/PlistBuddy -c \"Set %s %s\" %s" % (changeSchemeNameKey, val, infoPath)
    
    print("status: ", status)
    if findSchemeIndex == -1 or status == 256:
        print("修改失败--因为不存在, findSchemeIndex: " + str(findSchemeIndex) + " status: " + str(status))
        # 需要新增，新增的位置就是count的位置添加一个新对象
        index = totalCount
        # 添加字典
        dictKey = ":CFBundleURLTypes:%s" % (index)
        scriptAddDict = "/usr/libexec/PlistBuddy -c \"Add %s dict\" %s" % (dictKey, infoPath)
        print("scriptAddDict: " + scriptAddDict)
        os.system(scriptAddDict)
        # 添加数组
        arrayKey = ":CFBundleURLTypes:%s:CFBundleURLSchemes" % (index)
        scriptAddArray = "/usr/libexec/PlistBuddy -c \"Add %s array\" %s" % (arrayKey, infoPath)
        print("scriptAddArray: " + scriptAddArray)
        os.system(scriptAddArray)
        # 添加内容
        key1 = ":CFBundleURLTypes:%s:CFBundleURLName" % (index)
        key2 = ":CFBundleURLTypes:%s:CFBundleURLSchemes:0" % (index)
        key3 = ":CFBundleURLTypes:%s:CFBundleTypeRole" % (index)

        scriptStr11 = "/usr/libexec/PlistBuddy -c \"Add %s string %s\" %s" % (key1, schemeName, infoPath)
        scriptStr22 = "/usr/libexec/PlistBuddy -c \"Add %s string %s\" %s" % (key2, schemeUrl, infoPath)
        scriptStr33 = "/usr/libexec/PlistBuddy -c \"Add %s string %s\" %s" % (key3, "Editor", infoPath)
        print("scriptStr11: " + scriptStr11)
        print("scriptStr22: " + scriptStr22)
        print("scriptStr33: " + scriptStr33)

        os.system(scriptStr11)
        os.system(scriptStr22)
        os.system(scriptStr33)
        print("添加成功")
    else:
        print("执行成功")
```
----
完整demo: `test.py`
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import os
import sys

def readPlist2Json(plistPath):
    # iOS小技能：使用PlistBuddy/plutil进行增删改查plist文件
    # https://juejin.cn/post/7151969500424831012
    # 读取执行命令后的输出内容
    # plutil -p
    exeStr = "/usr/bin/plutil -p %s" % (plistPath)
    print("exeStr: " + exeStr)

    jsonStr = ""
    with os.popen(exeStr) as input_file: 
        linesList = input_file.readlines()
        totalCount = len(linesList)
        for i in range(0,totalCount):
            lineStr = linesList[i]
            if "=>" in lineStr:
                keyVList = lineStr.split('=>')
                keyStr = keyVList[0]
                valStr = keyVList[1]
                valStr = valStr.replace("\n", "")
                # 有key的行
                if "\" =>" in lineStr:
                    #key是String，属于dict的key
                    if " => [" in lineStr:
                        # value是一个array
                        jsonStr = jsonStr + keyStr + ":" + valStr + "\n"
                    elif " => {" in lineStr:
                        # value是一个dict
                        jsonStr = jsonStr + keyStr + ":" + valStr + "\n"
                    else:
                        # value是一个普通的值
                        jsonStr = jsonStr + keyStr + ":" + valStr + "," + "\n"
                else:
                    #属于Array的index
                    #key是String，属于dict的key
                    if " => [" in lineStr:
                        # value是一个array
                        jsonStr = jsonStr + valStr + "\n"
                    elif " => {" in lineStr:
                        # value是一个dict
                        jsonStr = jsonStr + valStr + "\n"
                    else:
                        # value是一个普通的值
                        jsonStr = jsonStr + valStr + "," + "\n"
            else:
                lineStr2 = lineStr.replace("\n", "")
                if i == totalCount - 1 :
                    jsonStr = jsonStr + lineStr2
                else:
                    if "]" in lineStr:
                        jsonStr = jsonStr + lineStr2 + "," + "\n"
                    elif "}" in lineStr:
                        jsonStr = jsonStr + lineStr2 + "," + "\n"
                    else:
                        jsonStr = jsonStr + lineStr2 + "\n"


    # 【python】str与json类型转换
    # https://blog.csdn.net/lluozh2015/article/details/75092877
    json1 = eval(jsonStr)
    # print("The type of json1 is: ", type(json1))
    return json1
    
def changeInfoPlistScheme(infoPath, schemeName, schemeUrl):
    # PlistBuddy基本使用方法
    # https://juejin.cn/post/6844903716395417614
    plistDict = readPlist2Json(plistPath)
    CFBundleURLTypes = plistDict["CFBundleURLTypes"]
    totalCount = len(CFBundleURLTypes)
    findSchemeIndex = -1
    for i in range(0,totalCount):
        schemeDict = CFBundleURLTypes[i]
        # print(schemeDict)
        CFBundleURLName = schemeDict["CFBundleURLName"]
        # 严格相等的判断
        # if CFBundleURLName == schemeName:
        #     findSchemeIndex = i
        #     break
        # 包含前缀的判断
        if schemeName in CFBundleURLName:
            findSchemeIndex = i
            break
    
    status = 0
    if findSchemeIndex != -1:
        changeSchemeUrlKey = ":CFBundleURLTypes:%s:CFBundleURLSchemes:0" % (findSchemeIndex)
        changeSchemeUrlExeStr = "/usr/libexec/PlistBuddy -c \"Set %s %s\" %s" % (changeSchemeUrlKey, schemeUrl, infoPath)
        print("changeSchemeUrlExeStr: " + changeSchemeUrlExeStr)
        status = os.system(changeSchemeUrlExeStr)
        # changeSchemeNameKey = ":CFBundleURLTypes:%s:CFBundleURLName" % (findSchemeIndex)
        # changeSchemeNameExeStr = "/usr/libexec/PlistBuddy -c \"Set %s %s\" %s" % (changeSchemeNameKey, val, infoPath)
    
    print("status: ", status)
    if findSchemeIndex == -1 or status == 256:
        print("修改失败--因为不存在, findSchemeIndex: " + str(findSchemeIndex) + " status: " + str(status))
        # 需要新增，新增的位置就是count的位置添加一个新对象
        index = totalCount
        # 添加字典
        dictKey = ":CFBundleURLTypes:%s" % (index)
        scriptAddDict = "/usr/libexec/PlistBuddy -c \"Add %s dict\" %s" % (dictKey, infoPath)
        print("scriptAddDict: " + scriptAddDict)
        os.system(scriptAddDict)
        # 添加数组
        arrayKey = ":CFBundleURLTypes:%s:CFBundleURLSchemes" % (index)
        scriptAddArray = "/usr/libexec/PlistBuddy -c \"Add %s array\" %s" % (arrayKey, infoPath)
        print("scriptAddArray: " + scriptAddArray)
        os.system(scriptAddArray)
        # 添加内容
        key1 = ":CFBundleURLTypes:%s:CFBundleURLName" % (index)
        key2 = ":CFBundleURLTypes:%s:CFBundleURLSchemes:0" % (index)
        key3 = ":CFBundleURLTypes:%s:CFBundleTypeRole" % (index)

        scriptStr11 = "/usr/libexec/PlistBuddy -c \"Add %s string %s\" %s" % (key1, schemeName, infoPath)
        scriptStr22 = "/usr/libexec/PlistBuddy -c \"Add %s string %s\" %s" % (key2, schemeUrl, infoPath)
        scriptStr33 = "/usr/libexec/PlistBuddy -c \"Add %s string %s\" %s" % (key3, "Editor", infoPath)
        print("scriptStr11: " + scriptStr11)
        print("scriptStr22: " + scriptStr22)
        print("scriptStr33: " + scriptStr33)

        os.system(scriptStr11)
        os.system(scriptStr22)
        os.system(scriptStr33)
        print("添加成功")
    else:
        print("执行成功")

print("changeScheme=====changeScheme")
print(sys.argv)
scriptPath = sys.argv[0]
plistPath = sys.argv[1]
# json1 = readPlist2Json(plistPath)
# CFBundleURLTypes = json1["CFBundleURLTypes"]
# print("CFBundleURLTypes len ", len(CFBundleURLTypes))
# ccc = json1["CFBundleURLTypes"]
# print(ccc)

changeInfoPlistScheme(plistPath, "ccc", "cccHello")
changeInfoPlistScheme(plistPath, "xxx", "xxxWorld")



# python ~/Desktop/test.py ~/Desktop/Info.plist
```
执行效果
```
 ~  python ~/Desktop/test.py ~/Desktop/Info.plist
changeScheme=====changeScheme
['/Users/xxx/Desktop/test.py', '/Users/xxx/Desktop/Info.plist']
exeStr: /usr/bin/plutil -p /Users/xxx/Desktop/Info.plist
changeSchemeUrlExeStr: /usr/libexec/PlistBuddy -c "Set :CFBundleURLTypes:1:CFBundleURLSchemes:0 cccHello" /Users/xxx/Desktop/Info.plist
('status: ', 0)
执行成功
exeStr: /usr/bin/plutil -p /Users/xxx/Desktop/Info.plist
('status: ', 0)
修改失败--因为不存在, findSchemeIndex: -1 status: 0
scriptAddDict: /usr/libexec/PlistBuddy -c "Add :CFBundleURLTypes:4 dict" /Users/xxx/Desktop/Info.plist
scriptAddArray: /usr/libexec/PlistBuddy -c "Add :CFBundleURLTypes:4:CFBundleURLSchemes array" /Users/xxx/Desktop/Info.plist
scriptStr11: /usr/libexec/PlistBuddy -c "Add :CFBundleURLTypes:4:CFBundleURLName string xxx" /Users/xxx/Desktop/Info.plist
scriptStr22: /usr/libexec/PlistBuddy -c "Add :CFBundleURLTypes:4:CFBundleURLSchemes:0 string xxxWorld" /Users/xxx/Desktop/Info.plist
scriptStr33: /usr/libexec/PlistBuddy -c "Add :CFBundleURLTypes:4:CFBundleTypeRole string Editor" /Users/xxx/Desktop/Info.plist
添加成功
```
![image](/assets/img/other/changeScheme.png)