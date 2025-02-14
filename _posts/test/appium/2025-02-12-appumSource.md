---
title: appium5：源码分析
author: 独孤流
date: 2025-02-12 12:05:00 +0800
categories: [autotest, appium]
tags: [autotest]     # TAG names should always be lowercase
---

参考：
- [appium 源码](https://github.com/appium/appium.git)
- [Appium介绍](https://appium.io/docs/en/latest/intro/)
- [appium-inspector](https://github.com/appium/appium-inspector)
- [appium-路径](https://github.com/appium/appium/blob/master/packages/base-driver/docs/mjsonwp/protocol-methods.md)
- [appium-路径2](https://github.com/appium/appium/blob/master/packages/base-driver/lib/protocol/routes.js)
- [appium-xcuitest-driver](https://github.com/appium/appium-xcuitest-driver.git)
- [appium-ios-device](https://github.com/appium/appium-ios-device.git)
以查看app是否已经安装为例

### 1、在inspector里或者直接使用postman调用
url: `http://127.0.0.1:4723/session/{sessionId}/appium/device/app_installed`
参数：`appId` 或 `bundleId`， 也就是app的`bundleId`
参数的方式： `x-www-form-urlencoded`

一、 使用appium请求的例子
```
import requests
# Appium 服务器的地址
appium_url =  "http://127.0.0.1:4723"
url = "{appium_url}/wd/hub/session/{sessionId}/appium/device/app_installed"
# 你的 appium生成的sessionId（假设已经创建了一个会话）
session_id = "47feb226-a7be-438b-b0c5-8cee47fad714"
url = url.format(appium_url=appium_url, sessionId=session_id)
# 请求的应用包名（例如对于 iOS 应用）
app_id = "com.example.app"
response = requests.post(url, data={"bundelId": "xx.xx.xx"})
# 打印响应内容
print(response.status_code)  # 输出状态码
print(response.json())  # 输出响应的 JSON 数据
```


### 二、直接请求WDA的方式
2.1、 请求激活app的例子
```
# WDA
wda_url = "http://192.168.50.141:8100"
# 创建会话
response = requests.post(f"{wda_url}/session", json={
    "capabilities": {
        "alwaysMatch": {
            "arguments": [],
            "environment": {}
        }
    }
})

# 获取 sessionId
session_id = response.json()["sessionId"]
print("Session ID:", session_id)

url = "{wda_url}/session/{sessionId}/wda/apps/activate"
# 这里替换 {sessionId} 为你的实际 sessionId
url = url.format(wda_url=wda_url, sessionId=session_id)
# json格式的请求
response = requests.post(url, json.dumps({"bundleId": bundleId}))
# 打印响应内容
print(response.status_code)  # 输出状态码
print(response.json())  # 输出响应的 JSON 数据
```

2.2 请求app列表的例子
```
# WDA
wda_url = "http://192.168.50.141:8100"
url = "{wda_url}/session/{sessionId}/wda/apps/list"
# 这里替换 {sessionId} 为你的实际 sessionId
url = url.format(wda_url=wda_url, sessionId=session_id)
# 发送请求
response = requests.get(url)

# 打印响应内容
print(response.status_code)  # 输出状态码
print(response.json())  # 输出响应的 JSON 数据
```
### 三、对app进行安装和卸载
调用卸载、安装app的流程

模拟器
https://github.com/appium/appium/tree/master
https://github.com/appium/appium-xcuitest-driver
https://github.com/appium/appium-ios-simulator
https://github.com/appium/node-simctl

真机
https://github.com/appium/appium/tree/master
https://github.com/appium/appium-xcuitest-driver
https://github.com/appium/appium-ios-device
调用各个服务
real-device.js 
services.startInstallationProxyService
AfcService
com.apple.crashreportcopymobile

调用真机或模拟器的device初始化
determineDevice()
const device = await getSimulator(this.opts.udid） {getSimulator} from 'appium-ios-simulator';
const device = new RealDevice(this.opts.udid, this.log); appium-ios-device

https://github.com/appium/appium-xcuitest-driver/blob/master/lib/simulator-management.js
-> https://github.com/appium/appium-ios-device
-> https://github.com/appium/appium-ios-simulator/blob/master/lib/extensions/applications.js
-> https://github.com/appium/node-simctl/blob/master/lib/simctl.js
-> https://github.com/appium/node-simctl/blob/master/lib/subcommands/uninstall.js
-> this.exec('uninstall',...)
-> https://github.com/appium/node-simctl/blob/master/lib/simctl.js -> 'simctl xxxx' 
-> `xcrun simctl xxxxx`
`xcrun simctl uninstall booted com.example.myapp`
`xcrun simctl uninstall ABCD-1234-5678-90EF com.example.myapp`
```
# 操作模拟器使用 xcrun simctl
# 卸载app
xcrun simctl uninstall udid的值 bundleId的值
# 安装app
xcrun simctl install udid的值 xxx.app的地址
```

安装同理
-> https://github.com/appium/node-simctl/blob/master/lib/subcommands/install.js
-> this.exec('install',...)
-> https://github.com/appium/node-simctl/blob/master/lib/simctl.js -> 'simctl xxxx' 
`xcrun simctl install ABCD-1234-5678-90EF /xx/xx.ipa|app`

流程：
```
[HTTP] --> POST /session/4efedc27-f9c1-423b-a24b-563bd678679a/appium/device/remove_app {"appId":"xx.xx.xx"}
[XCUITestDriver@09b7] Uninstalling the application with bundle identifier 'xx.xx.xx' from the Simulator with UDID '9D5EA427-C4F4-49B3-90D5-4FB1F75554A8'
```
`[https://github.com/appium/appium-xcuitest-driver/blob/master/lib/driver.js` -> `async executeCommand(cmd, ...args)` 
-> `https://github.com/appium/appium-xcuitest-driver/blob/master/lib/commands/app-management.js` -> `async mobileRemoveApp(bundleId)`
-> `https://github.com/appium/appium-ios-device`
-> 
### 四、如果电脑（Mac或pc都可以）安装了go-ios，直接使用go-ios执行命令

这使得PC上的管理工具（例如 iTunes、libimobiledevice、go‑ios 等）能够通过建立与设备之间的通信连接，发送命令来调用和控制这些服务的功能。


```
# go-ios支持的相关命令
# https://github.com/danielpaulus/go-ios
# https://github.com/danielpaulus/go-ios/blob/main/main.go
ios install /xx/xx/xxx.ipa
ios uninstall xx.xx.xx

# 由于iPhone与电脑（pc或mac）使用usb连接后，利用USBMMUX协议来与iPhone建立连接并通过lockdownd服务通信
# 通信的命令比如调用com.apple.mobile.installation_proxy，并卸载、安装app
```

建立隧道命令流程\
main.go ->
-> `device, err := ios.GetDevice(udid)` (建立链接)
-> 等待接收命令 
-> `b, _ = arguments.Bool("tunnel")` （如果命令是启动通道app）
-> `startTunnel`
-> `pairRecordsPath, _ := arguments.String("--pair-record-path")`
-> `startTunnel(context.TODO(), pairRecordsPath, tunnelInfoPort, useUserspaceNetworking)`
-> `tm := tunnel.NewTunnelManager(pm, userspaceTUN)`
-> `func startTunnel(ctx context.Context, recordsPath string, tunnelInfoPort int, userspaceTUN bool) {`
-> `p.StartProcess(bundleID, env, []interface{}{}, opts)` (执行安装命令)
-> `func (p ProcessControl) StartProcess(bundleID string, envVars map[string]interface{}, ` 
-> `pm, err := tunnel.NewPairRecordManager(recordsPath)`



启动app流程\
main.go ->
-> `device, err := ios.GetDevice(udid)` (建立链接)
-> 等待接收命令 
-> `b, _ = arguments.Bool("launch")` （如果命令是启动app）
-> `pControl, err := instruments.NewProcessControl(device)` processcontrol.go开启服务"com.apple.dt.Xcode.WatchProcessControl"）
-> `pid, err := pControl.LaunchAppWithArgs(bundleID, args, envs, opts)` (调用服务执行卸载命令)
processcontrol.go
-> `func (p *ProcessControl) LaunchApp(bundleID string, my_opts map[string]any) (uint64, error) {`
-> `p.StartProcess(bundleID, env, []interface{}{}, opts)` (执行安装命令)
-> `func (p ProcessControl) StartProcess(bundleID string, envVars map[string]interface{}, ` 
-> `msg, err := p.processControlChannel.MethodCall("launchSuspendedProcessWithDevicePath:bundleIdentifier:environment:arguments:options:",...)`


安装app流程\
main.go ->
-> `device, err := ios.GetDevice(udid)` (建立链接)
-> 等待接收命令 
-> `b, _ = arguments.Bool("install")` （如果命令是安装app）
-> `installApp(device, path)` (调用卸载方法)
-> `conn, err := zipconduit.New(device)` （Zipconduit_installer.go开启服务 "com.apple.streaming_zip_conduit.shim.remote"）
-> `err = conn.SendFile(path)` (调用服务执行卸载命令)
（Zipconduit_installer.go
-> `func (conn Connection) SendFile(appFilePath string) error {`
-> `conn.sendDirectory(appFilePath) or conn.sendIpaFile(appFilePath)` (执行安装命令)
-> `conn.deviceConn.Write` 文件写入

卸载app流程\
main.go ->
-> `device, err := ios.GetDevice(udid)` (建立链接)
-> 等待接收命令 
-> `b, _ = arguments.Bool("uninstall")` （如果命令是卸载app）
-> `uninstallApp(device, bundleID)` (调用卸载方法)
-> `svc, err := installationproxy.New(device)` （installationproxy.go开启服务 "com.apple.mobile.installation_proxy"）
-> `svc.Uninstall(bundleId)` (调用服务执行卸载命令)
installionproxy.go
-> `func (c *Connection) Uninstall(bundleId string) error`
-> `b = xxxx; c.deviceConn.Send(b)` (组装命令，执行命令)


----

