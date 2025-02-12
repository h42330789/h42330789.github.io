---
title: appium4：python完整执行一个login的测试
author: 独孤流
date: 2025-02-11 12:05:00 +0800
categories: [autotest, appium]
tags: [autotest]     # TAG names should always be lowercase
---

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 用于获取参数
import sys
# 打印日志
import logging

# Appium相关
from appium import webdriver
from appium.options.ios import XCUITestOptions
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.actions.action_builder import ActionBuilder
from selenium.webdriver.common.actions.interaction import POINTER_TOUCH
from selenium.webdriver.common.actions.pointer_input import PointerInput

# 计算时间，sleep相关
import time

# 图片截取相关
from PIL import Image
import io
import base64

# 图片识别相关
import cv2
import numpy as np

# 随机数
import random

# 配置 logging，设置日志格式包括时间
logging.basicConfig(format='%(asctime)s - %(message)s', level=logging.INFO)

# 检测安装及重新运行
def check_and_launch_app(driver, bundleId, app):
    try:
        # 检测是否安装了app
        is_app_installed = driver.is_app_installed(bundleId)
        if is_app_installed:
            # 已经安装，杀掉app，重启app
            # 卸载app
            driver.remove_app(bundleId)
            # 安装app
            driver.install_app(app)
            # 杀掉APP进程
            # driver.terminate_app(bundleId)
        else:
            # 没有安装APP，安装
            driver.install_app(app)
        # 激活已安装的应用（如果它已经运行）
        logging.error(f"Activating app with bundleId: {bundleId}")
        # 启动app
        driver.activate_app(bundleId)
    except Exception as e:
        logging.error(f"Error while activating app: {e}")
        logging.error("Launching app...")
        driver.launch_app()

# 通知弹窗
def testNotifyAlert(driver):
    # 等待并点击第二个单元格
    try:
        # 系统弹窗
        alert = driver.switch_to.alert
        if alert:
            alert.accept()  # 点击 OK 或确认
            logging.error("点击了alert")
        else:
            logging.error("没有系统alert")
    except Exception as e:
        # logging.error(f"Error1: {e}")
        logging.error(f"alert 不存在")

# 欢迎页面
def testWelcomPage(driver):
    try:

        # 尝试获取 contentSize 属性
        # content_size = scroll_view.get_attribute("contentSize")
        # 打印出 contentSize
        # print(f"ScrollView contentSize: {content_size}")
        welcomeStartBtn = WebDriverWait(driver, 2.5).until(
            EC.element_to_be_clickable((By.XPATH, 'xxxxx'))
        )
        # logging.error(f"win.size {driver.get_window_size()}")
        win_width = driver.get_window_size().get('width')
        win_height = driver.get_window_size().get('height')
        action_builder = ActionBuilder(driver, mouse=PointerInput(POINTER_TOUCH, "touch"))
        # # 执行 swipe 操作
        actions = action_builder.pointer_action
        actions.move_to_location(win_width, win_height/2).pointer_down()
        actions.move_to_location(0, win_height/2).pointer_up()
        action_builder.perform()
        time.sleep(0.5)

        # driver.swipe(0, 0, win_width, 0, 1000)  # 持续 1000 毫秒

        actions = action_builder.pointer_action
        actions.move_to_location(win_width, win_height/2).pointer_down()
        actions.move_to_location(0, win_height/2).pointer_up()
        action_builder.perform()
        time.sleep(0.5)

        # 通过id 查找元素
        # welcomeStartBtn = driver.find_element(By.ACCESSIBILITY_ID, "xxxx")
        # welcomeStartBtn = driver.find_element(By.XPATH, 'xxxx')
        if welcomeStartBtn:
            welcomeStartBtn.click()
            logging.error("进入登录页面!")
        else:
            logging.error("没有马上开始的按钮")
        
    except Exception as e:
        logging.error(f"Error2: {e}")
        logging.error(f"马上开始按钮 不存在")

# 登录登录页面
def testLoginPage(driver):
    try:
        # 通过xpath获取元素
        # 进入登录页面后点击邮箱登录
        emailPage = WebDriverWait(driver, 0.5).until(
            EC.element_to_be_clickable((By.XPATH, 'xxxx'))
        )
        # 1、点击找到的单元格
        emailPage.click()
        logging.error("点击邮箱登录!")

        # 2、找到邮箱输入框，输入邮箱
        emailInput = WebDriverWait(driver, 0.5).until(
            EC.element_to_be_clickable((By.XPATH, 'xxx'))
        )
        emailInput.send_keys("a1@test.com")
        logging.error("在邮箱输入框里输入a5@test.com")

        # 3、找到获取验证码按钮，点击
        sendEmailCodeBtn = WebDriverWait(driver, 1.5).until(
            EC.element_to_be_clickable((By.XPATH, 'xxxx'))
        )
        sendEmailCodeBtn.click()
        logging.error("点击发送验证码")
    
        # 4、滑动验证
        checkSlider(driver)

        # 5、 输入验证码
        smsInput = WebDriverWait(driver, 20).until(
            EC.element_to_be_clickable((By.XPATH, 'xxxx'))
        )
        smsInput.send_keys("xxxx")
        logging.error("在验证码输入框里输入xxx")

        # 6、 点击登录按钮
        loginBtn = WebDriverWait(driver, 20).until(
            EC.element
        )

# 滑块检测
def checkSlider(driver, checkCount = 10, checkIndex = 0):
    logging.error(f"checkSlider-start checkIndex:{checkIndex+1} / {checkCount}")
    # 4、找到拖拽控件，拖拽
    win_size = driver.get_window_size()
    logging.error(f"win_size: {win_size}")
    # 4.1 找到整个app截取成图片
    # 截取整个页面的截图
    # 获取截图之前等待几秒钟
    time.sleep(2)  # 延迟 2 秒，可以根据需要调整延迟时间
    logging.error(f"checkSlider-sleep结束，开始截屏")
    screenshot = driver.get_screenshot_as_base64()
    if screenshot:
        logging.error("页面截图")
        screenshot_image = Image.open(io.BytesIO(base64.b64decode(screenshot)))
        screenshot_width, screenshot_height = screenshot_image.size
        logging.error(f"screenshot_image.size: {screenshot_image.size}")
        scale_x = screenshot_width / win_size['width']
        logging.error(f"scale_x: {scale_x}")
        # 计算新的宽度和高度，按 scale_x 压缩
        new_width = int(screenshot_width / scale_x)  # 强制转换为整数
        new_height = int(screenshot_height / scale_x)  # 强制转换为整数
        screenshot_image = screenshot_image.resize((new_width, new_height), Image.Resampling.LANCZOS)
        scale_x = 1.0 #screenshot_width / win_size['width']
        logging.error(f"scale_x: {scale_x}")
        # scale_x = round(scale_x)
        # logging.error(f"四舍五入: scale_x: {scale_x}")
    else:
        logging.error("页面截图 不存在")

    logging.error(f"checkSlider-开始查找元素")
    # 4.1 找到容器
    drag_container = WebDriverWait(driver, 5).until(
        EC.element_to_be_clickable((By.XPATH, 'xxxxxxx'))
    )
    logging.error(f"查找到drag容器")
    

    # 4.2 找到滑块
    drag_item_element = drag_container.find_elements(By.XPATH, "./*")[1]
    item_width = drag_item_element.size['width']
    # # 裁剪并保存元素的图片 offsetX = 0, offsetWidth = 0, offsetHeight = 0
    drag_item_image = cropImage(scale_x, scale_x, screenshot_image, drag_item_element, "drag_image.png", 10, -20 , 20)
    logging.error(f"查找到drag-item")

    # 4.3 找到背景
    drag_bg_element = drag_container.find_elements(By.XPATH, "./*")[2]
    # 裁剪并保存元素的图片
    cropImage(scale_x, scale_x, screenshot_image, drag_bg_element, "drag_full_bg.png")
    logging.error(f"查找到drag-bg")

    # # 裁剪并保存元素的图片
    drag_bg_image = cropImage(scale_x, scale_x, screenshot_image, drag_bg_element, "drag_bg.png", item_width, -item_width+10, 0)
    logging.error(f"查找到drag-bg")

    
    

    # # 4.4 使用opencv识别位置
    offsetx = calculate_slide_distance(drag_bg_image, drag_item_image)
    logging.error(f"计算图片滑动位置 {offsetx}")
    # logging.error(f"计算图片滑动位置BBBBB {offsetx3}")
    offsetx = offsetx/scale_x
    logging.error(f"scale后的滑动位置 {offsetx}")
   
    item_width = drag_item_element.size['width']
    offsetx = offsetx + item_width
    logging.error(f"+item_width {item_width} offsetx:{offsetx}")
    # random_number = random.randint(1, 5)
    # offsetx = offsetx + random_number
    # logging.error(f"scale+random_number {random_number}后的滑动位置 {offsetx}")

    bg_width = drag_bg_element.size['width']
    logging.error(f"滑动距离 {offsetx} / {bg_width}")

   
    smooth_slide(driver, drag_item_element, offsetx)
    
    if checkIndex >= checkCount:
        # 检测数量超过指定数量，不再循环判断
        logging.error(f"重试次数用完")
        return
    # 延迟0.5秒后计算是否成功，也就是能否找到数据
    time.sleep(2)  # 等待 2 秒
    try:


# 移动滑块
def smooth_slide(driver, slider, offsetx, offsety=0, steps=100):

    # 获取滑块的初始位置和大小
    slider_location = slider.location
    slider_size = slider.size

    # 计算滑块的起始点和目标点
    start_x = slider_location['x'] + slider_size['width'] / 2
    start_y = slider_location['y'] + slider_size['height'] / 2
    end_x = start_x + offsetx  # 根据需要滑动的距离调整
    end_y = start_y

    # 创建 PointerInput 对象
    pointer_input = PointerInput(POINTER_TOUCH, "touch")

    # 创建 ActionBuilder 对象
    action_builder = ActionBuilder(driver, mouse=pointer_input)

    # 定义动作链
    actions = action_builder.pointer_action

    # 按下滑块
    actions.move_to_location(start_x, start_y).pointer_down()

    # 滑动总距离
    total_distance = end_x - start_x

    # 滑动参数
    steps = 8  # 总步数
    initial_delay = 0.005  # 初始延迟时间
    final_delay = 0.001 # 最终延迟时间
    delay_decrease = (initial_delay - final_delay) / steps  # 延迟时间递减量

    # 滑动
    for i in range(steps):
        # 计算当前步的移动距离
        step_distance = total_distance / steps
        # 计算当前步的目标位置
        target_x = start_x + step_distance * (i + 1)
        # 移动滑块
        actions.move_to_location(target_x, start_y)
        if i < 7:
            # 动态调整延迟时间，模拟加速效果
            stime = max(0.01 - 0.001 * i, 0.001)
            time.sleep(stime)
        else:
            stime = max(0.05 + 0.01 * i, 0.001)
            time.sleep(stime)

    # 释放滑块
    actions.pointer_up()

    # 执行动作
    action_builder.perform()

# 截取图片
def cropImage(scale_x, scale_y, screenshot, element, name, offsetX = 0, offsetWidth = 0, offsetHeight = 0):
    # 获取元素的位置和大小
    location = element.location
    size = element.size
    # logging.error(f"location: {location}")
    # logging.error(f"size: {size}")
    # logging.error(f"offsetX: {offsetX}")
    offsetX2 = offsetX*scale_x
    # logging.error(f"offsetX-scale: {offsetX2}")

    # 将截图从 Base64 解码为图像
    # image = Image.open(io.BytesIO(base64.b64decode(screenshot)))
    image = screenshot
    # image = Image.open(screenshot)
    element_x = location['x']
    element_y = location['y']
    element_width = size['width']
    element_height = size['height']
    
    screenshot_x = int(element_x * scale_x) + offsetX2
    screenshot_y = int(element_y * scale_y)
    screenshot_width = int(element_width * scale_x) + offsetWidth * scale_x
    screenshot_height = int(element_height * scale_y) + offsetHeight * scale_x

    # 计算裁剪区域
    left = screenshot_x
    top = screenshot_y
    right = screenshot_x + screenshot_width
    bottom = screenshot_y + screenshot_height
    # logging.error(f"left: {left}")
    # logging.error(f"top: {top}")
    # logging.error(f"right: {right}")
    # logging.error(f"bottom: {bottom}")


    # 裁剪并保存元素的图片
    element_image = image.crop((left, top, right, bottom))
    logging.error(f"截取图片")
    # 保存图片，不是必须
    # element_image.save(name)
    # logging.error(f"保存图片")
    return element_image

# 计算图片距离
def calculate_slide_distance(background_image, slider_image):
    # 将 PIL Image 对象转换为 OpenCV 格式的图像
    background = np.array(background_image)
    slider = np.array(slider_image)

    # OpenCV 使用 BGR 格式，需要转换为灰度图
    background_gray = cv2.cvtColor(background, cv2.COLOR_RGB2GRAY)  # PIL -> OpenCV
    slider_gray = cv2.cvtColor(slider, cv2.COLOR_RGB2GRAY)  # PIL -> OpenCV

    # 使用Canny边缘检测来突出显示图像中的边缘
    background_edges = cv2.Canny(background_gray, 100, 200)  # 你可以调整阈值
    slider_edges = cv2.Canny(slider_gray, 100, 200)

    # 使用模板匹配找到滑块图像在背景图中的位置
    result = cv2.matchTemplate(background_edges, slider_edges, cv2.TM_CCOEFF_NORMED)

    # 获取最大匹配值的位置
    min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)

    # max_loc 是模板匹配中最大相似度位置的左上角坐标
    logging.error(f"Max matching location (x, y): {max_loc}")

    # 计算滑动的水平距离
    slide_distance = background.shape[1] - (max_loc[0] + slider.shape[1])

    logging.error(f"Calculated slide distance: {slide_distance}")
    
    # 可视化匹配位置，绘制矩形框显示匹配结果
    top_left = max_loc
    bottom_right = (top_left[0] + slider.shape[1], top_left[1] + slider.shape[0])
    cv2.rectangle(background, top_left, bottom_right, (0, 255, 0), 2)

    # 标记背景图中阴影部分
    # # 在滑块的阴影部分使用矩形框或其他标记
    # shadow_top_left = (max_loc[0] - 10, max_loc[1] + slider.shape[0])  # 阴影的上边缘坐标，稍微往下调整
    # shadow_bottom_right = (max_loc[0] + slider.shape[1] + 10, max_loc[1] + slider.shape[0] + 20)  # 阴影的下边缘坐标
    # cv2.rectangle(background, shadow_top_left, shadow_bottom_right, (255, 0, 0), 2)  # 红色标记阴影区域

    # 叠加两张图像以显示重叠效果
    # # 将滑块图像粘贴到背景图像上
    # overlapped_image = background.copy()
    # slider_resized = cv2.resize(slider, (slider.shape[1], slider.shape[0]))  # 如果需要调整大小
    # overlapped_image[top_left[1]:bottom_right[1], top_left[0]:bottom_right[0]] = slider_resized

    # 保存背景图、重叠图像

# 开启服务
def start_appium_session(udid, app, bundleId):
    # 配置 Appium 和设备连接
    options = XCUITestOptions().load_capabilities({
        'platformName': 'iOS',
        'automationName': 'XCUITest',
        'udid': udid,  # 设备 UDID
        'noReset': True,
        'usePrebuiltWDA': True,
    })

    # 启动 Appium 会话
    driver = webdriver.Remote(
        'http://127.0.0.1:4723',
        options=options,
        direct_connection=True
    )
    # 如果app在运行就杀掉app重新进行
    check_and_launch_app(driver, bundleId, app)
    testNotifyAlert(driver)
    testWelcomPage(driver)
    testLoginPage(driver)
    testChangeTradePwd(driver)
    testMinePage(driver)
    testContactsPage(driver)
    # 关闭会话
    driver.quit()

import plistlib
import os
def get_app_info(app_path):
    plist_path = os.path.join(app_path, 'Info.plist')
    logging.error(f"plist_path: {plist_path}")
    
    # 检查 Info.plist 是否存在
    if not os.path.exists(plist_path):
        raise FileNotFoundError(f"{plist_path} not found.")
    
    try:
        # 读取 plist 文件
        with open(plist_path, 'rb') as file:
            plist = plistlib.load(file)
        # logging.error(f"plist: {plist}")
        # 提取 bundleId 和 app 名称
        bundle_id = plist.get('CFBundleIdentifier', 'Unknown')  # 默认值 'Unknown'
        app_name = plist.get('CFBundleName', 'Unknown')  # 默认值 'Unknown'
        logging.error(f"bundle_id: {bundle_id}")
        logging.error(f"app_name: {app_name}")
        return bundle_id, app_name
    
    except Exception as e:
        print(f"Error reading plist: {e}")
        return None, None

# 仅当脚本作为主程序运行时，才会执行下面的代码
if __name__ == "__main__":

    # 检查是否有足够的参数
    if len(sys.argv) < 3:
        logging.error("Usage: python test1.py <udid> <file>")
        sys.exit(1)

    # 获取传递的参数
    scriptPath = sys.argv[0]
    udid = sys.argv[1]
    app = sys.argv[2]

    logging.error(f"scriptPath: {scriptPath}")
    logging.error(f"udid: {udid}")
    logging.error(f"app: {app}")

    bundleId, appName = get_app_info(app)
    

    # 打印参数或者执行其他操作
    
    logging.error(f"bundleId: {bundleId}")
    logging.error(f"appName: {appName}")

    start_appium_session(udid, app, bundleId)



# 安装Python相关依赖
# 创建虚拟环境安装
# python3 -m venv venv
# source venv/bin/activate
# 安装相关依赖
# pip3 install Appium-Python-Client
# pip3 install selenium

# 安装PIL / image  No module named 'PIL'
# pip3 install Pillow
#  No module named 'cv2'
# pip3 install opencv-python

# 启动Appium
# appium --log-level debug --allow-cors
# 运行WDA
# xcodebuild -project "/xxx/xxx/WebDriverAgent.xcodeproj" -scheme WebDriverAgentRunner -destination "id=xxxx" test
# 或者使用go-ios启动安装在设备上的wda

# python3 /xxx/xxx/login.py "xxxx"  "/xxx/xxx/xxx.app"








```