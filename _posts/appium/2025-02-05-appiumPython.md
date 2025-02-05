---
title: appium-python脚本测试
author: 独孤流
date: 2025-02-04 00:05:00 +0800
categories: [Appium自动化测试]
tags: [appium]     # TAG names should always be lowercase
---

使用Python进行测试
参考：
- [python-client](https://github.com/appium/python-client)
- [webdriver-test](https://github.com/appium/python-client/tree/master/test/unit/webdriver)

```
# 安装Python相关依赖
# 创建虚拟环境安装
python3 -m venv venv
source venv/bin/activate
# 安装相关依赖
pip3 install Appium-Python-Client
pip3 install selenium
```

查找出相关的udid
```
# 获取模拟器设备列表，找到udid
xcrun simctl list devices | grep -E "Booted|Available|Shutdown"
# 获取真机查看方式，查看真是失败
idevice_id -l | head -n1
# 获取真机和模拟器查看的方式
xcrun xctrace list devices
```

创建一个python文件`test.py`
```
from appium import webdriver
from appium.options.ios import XCUITestOptions
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# 模拟器需要从编译的模拟器里拿
# /xxx/xx/Library/Developer/Xcode/DerivedData/xxx/Build/Products/Debug-iphonesimulator/xxx.app
# 真机要从真机里拿
# /xxx/xx/Library/Developer/Xcode/DerivedData/xxx/Build/Products/Debug-iphoneos/xxx.app
# 设置自定义选项并加载能力
options = XCUITestOptions().load_capabilities({
    'platformName': 'iOS',
    'platformVersion': '17.0',
    'deviceName': 'iPhone 15 Pro',
    'app': '/Users/macbookpro/Documents/study/studyAppium/UICatalog.app',
    'automationName': 'XCUITest',
    'noReset': True
})
# 也可以直接找到某个模拟器的udid或真机的udid进行测试
# options = XCUITestOptions().load_capabilities({
#    'platformName': 'iOS',
#    'udid': 'xxxxxxxxx',
#    'app': '/Users/macbookpro/Documents/study/studyAppium/UICatalog.app',
#    'automationName': 'XCUITest',
#    'noReset': True
# })
# 真机
#options = XCUITestOptions().load_capabilities({
#    'platformName': 'iOS',
#    "udid": "xxxxx",
#    'app': '/Users/macbookpro/Documents/study/studyAppium/test.ipa',
#    'automationName': 'XCUITest',
#    'noReset': True
#})
# 真机和模拟器的区别，模拟器可以安装任何bundleId直接运行代码，真机需要证书
# udid，真机或获取，模拟器也可以获取，但可以使用其他方式代替
# app, 真机需要真机的架构，模拟器需要模拟器的架构


# 启动 Appium 会话
driver = webdriver.Remote(
    'http://127.0.0.1:4723',
    options=options,
    direct_connection=True
)

# 等待并点击第二个单元格
try:
    # 显式等待直到表格单元格可点击
    cell = WebDriverWait(driver, 20).until(
        EC.element_to_be_clickable((By.XPATH, '//XCUIElementTypeTable/XCUIElementTypeCell[1]'))
    )
    # 点击找到的单元格
    cell.click()
    print("Cell clicked!")
except Exception as e:
    print(f"Error: {e}")



# python3 /xxx/xxx/xx/test.py

```