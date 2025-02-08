---
title: appium3：python执行测试脚本
author: 独孤流
date: 2025-02-04 12:05:00 +0800
categories: [autotest, appium]
tags: [autotest]     # TAG names should always be lowercase
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



配置参数说明
> `'platformName': 'iOS'`: （必须）平台，可填`iOS`, `tvOS`, `macOS`, `Windows`, `Android`测试iPhone固定填写`iOS`\
> `'automationName': 'XCUITest'`: （必须）测试框架，测试iPhone固定填写`XCUITest`\
> `'platformVersion': '17.0'`: （可选）测试的系统版本，一般测模拟器时配合使用，真机必须要，真机使用udid作为唯一\
> `'deviceName': 'iPhone 15 Pro'`: （可选）测试的模拟器的名称，一般测模拟器时配合使用，真机必须要，真机使用udid作为唯一\
> `'usePrebuiltWDA': True`: （可选）是否要保留测试设备上之前安装的DWA，建议配上True，不然还需要安装上才能使用\ .
> `'app': 'xxx.app'`: 可选）,要测试的APP，可以是`、/xx/xxx.ipa`、`/xx/xxx.app`、`xx.xx.xx`(bundleId)，填写了会默认安装或打开这个APP

在网页上查找元素的方式
https://inspector.appiumpro.com/
```
{
"platformName": "iOS",
"automationName": "XCUITest",
"platformVersion": "17.0",
"deviceName": "iPhone 15 Pro",
"usePrebuiltWDA": true
}
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
    'automationName': 'XCUITest',
    'platformVersion': '17.0',
    'deviceName': 'iPhone 15 Pro',
    'usePrebuiltWDA': True,
    'noReset': True,
    'app': '/Users/macbookpro/Documents/study/studyAppium/UICatalog.app'
})
# 真机或模拟器
# options = XCUITestOptions().load_capabilities({
#    'platformName': 'iOS',
#    'udid': 'xxxxxxxxx',
#    'automationName': 'XCUITest',
#    'usePrebuiltWDA': True,
#    'noReset': True,
#    'app': '/Users/macbookpro/Documents/study/studyAppium/UICatalog.app'
# })

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