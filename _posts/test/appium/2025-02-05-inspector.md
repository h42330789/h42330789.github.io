---
title: appium2：inspector的使用
author: 独孤流
date: 2025-02-04 10:05:00 +0800
categories: [autotest, appium]
tags: [autotest]     # TAG names should always be lowercase
---

### 一、配置启动
要使用inspector，这里主要使用web版本的`inspector`进行查看演示，免安装
地址：`https://inspector.appiumpro.com/`

前提：\
1、安装`appium`

2、启动`appium`
```
appium --log-level debug --allow-cors
```
![image](/assets/img/test/appium/appium_start.png)

3、打开地址`https://inspector.appiumpro.com/`
![image](/assets/img/test/appium/inspector1.png)
输入如下内容,并保存,系统及设备信息可修改
```
{
"platformName": "iOS",
"automationName": "XCUITest",
"platformVersion": "17.0",
"deviceName": "iPhone 15 Pro",
"usePrebuiltWDA": true
}
```
![image](/assets/img/test/appium/inspector2x.png)
![image](/assets/img/test/appium/inspector3.png)
点击`start session`就可正常链接了

### 二、查看元素节点
启动session后看到的第一个就是元素信息
![image](/assets/img/test/appium/inspector_4_1_source.png)

----
### 三、借用inspector配Python
链接信息参：
![image](/assets/img/test/appium/inspector_4_5_session.png)

不同的语言可以直接切换，然后就有相关的demo了
```
# This sample code supports Appium Python client >=2.3.0
# pip install Appium-Python-Client
# Then you can paste this into a file and simply run with Python

from appium import webdriver
from appium.options.common.base import AppiumOptions
from appium.webdriver.common.appiumby import AppiumBy

# For W3C actions
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.actions import interaction
from selenium.webdriver.common.actions.action_builder import ActionBuilder
from selenium.webdriver.common.actions.pointer_input import PointerInput

options = AppiumOptions()
options.load_capabilities({
	"platformName": "iOS",
	"appium:automationName": "XCUITest",
	"appium:platformVersion": "17.0",
	"appium:deviceName": "iPhone 15 Pro",
	"appium:usePrebuiltWDA": True,
	"appium:includeSafariInWebviews": True,
	"appium:newCommandTimeout": 3600,
	"appium:connectHardwareKeyboard": True
})

driver = webdriver.Remote("http://127.0.0.1:4723", options=options)


driver.quit()
```

### 四、操作相关指令及操作录制
![image](/assets/img/test/appium/inspector_4_3_recoard1.png)
![image](/assets/img/test/appium/inspector_4_0_home.png)
![image](/assets/img/test/appium/inspector_4_2_command.png)
![image](/assets/img/test/appium/inspector_4_3_recoard2.png)
相关操作的录制demo的内容
```
# This sample code supports Appium Python client >=2.3.0
# pip install Appium-Python-Client
# Then you can paste this into a file and simply run with Python

from appium import webdriver
from appium.options.common.base import AppiumOptions
from appium.webdriver.common.appiumby import AppiumBy

# For W3C actions
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.actions import interaction
from selenium.webdriver.common.actions.action_builder import ActionBuilder
from selenium.webdriver.common.actions.pointer_input import PointerInput

options = AppiumOptions()
options.load_capabilities({
	"platformName": "iOS",
	"appium:automationName": "XCUITest",
	"appium:platformVersion": "17.0",
	"appium:deviceName": "iPhone 15 Pro",
	"appium:usePrebuiltWDA": True,
	"appium:includeSafariInWebviews": True,
	"appium:newCommandTimeout": 3600,
	"appium:connectHardwareKeyboard": True
})

driver = webdriver.Remote("http://127.0.0.1:4723", options=options)
# 进入home
driver.execute_script('mobile:pressButton', {"name": "home"})
# 获取判断是否安装APP
is_app_installed = driver.is_app_installed('com.example.apple-samplecode.UICatalog')
# 安装APP
driver.install_app('/Users/macbookpro/Documents/study/studyAppium/UICatalog.app')
# 激活APP
driver.activate_app('com.example.apple-samplecode.UICatalog')
# 进入后台
driver.execute_script('mobile: backgroundApp', {"seconds": 2})
# 杀掉APP进程
driver.terminate_app('com.example.apple-samplecode.UICatalog')
# 移除APP
driver.remove_app('com.example.apple-samplecode.UICatalog')

driver.quit()
```

### 五、操作APP
5.0 获取系统弹出并操作
- [WebDriver](https://selenium-python.readthedocs.io/api.html#selenium.webdriver.remote.webdriver.WebDriver)
- [Alert](https://selenium-python.readthedocs.io/api.html#selenium.webdriver.common.alert.Alert)
- [docs](https://selenium-python.readthedocs.io/)
```
# 比如推送弹出，网络权限弹窗等各种弹窗
alert = driver.switch_to.alert
if alert:
    alert.accept()  # 点击 OK 或确认

# 截屏成base64字符串
screenshot_base64 = driver.get_screenshot_as_base64()
screenshot_bytes = driver.get_screenshot_as_png()
win_pos = driver.get_window_position()
win_size = driver.get_window_size()
# 查看元素结构
page_source = driver.page_source
logging.error(f"page_source {page_source}")
```

5.1 通过id获取元素
```
# 通过id查找元素，前提是要在代码里设置了ACCESSIBILITY_ID
# swift源码里 let loginButton = UIButton()
# swift源码里 loginButton.accessibilityIdentifier = "loginButton"
loginBtn = driver.find_element(By.ACCESSIBILITY_ID, "loginButton")
# 点击操作
loginBtn.click()
```

5.2 通过path获取元素
- [W3C的操作相关ActionChains](https://selenium-python.readthedocs.io/api.html#selenium.webdriver.common.action_chains.ActionChains.click)
```
# 通过path的方式获取，path使用inspector的source面板里获取
loginBtn = WebDriverWait(driver, 20).until(
    EC.element_to_be_clickable((By.XPATH, '//XCUIElementTypeButton[@name="登录"]'))
)
loginBtn.click()

nameInput = WebDriverWait(driver, 20).until(
    EC.element_to_be_clickable((By.XPATH, '//XCUIElementTypeTextField[@value="请输入姓名"]'))
)
nameInput.send_keys("fobe")

slide_item = WebDriverWait(driver, 20).until(
    EC.element_to_be_clickable((By.XPATH, '//XCUIElementTypeOther[@name="DEFAULT" and @label="DEFAULT"]'))
)
# 获取元素的位置
print(f"slide_item.location.x: {slide_item.location['x']}")
print(f"slide_item.location.y: {slide_item.location['y']}")
print(f"slide_item.size.width: {slide_item.size['width']}")
print(f"slide_item.size.height: {slide_item.size['height']}")
offsetx = 50
# slide_item水平方向滑动100的距离
ActionChains(driver).click_and_hold(slide_item).move_by_offset(offsetx, 0).release().perform()
```

