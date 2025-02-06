---
title: VPN解除特定域名代理
author: 独孤流
date: 2025-02-04 01:04:00 +0800
categories: [AI]
tags: [AI,ChatGPT,Deepseek]     # TAG names should always be lowercase
---

> ### 前言
> 由于电脑本身处于非限制区域IP，可以直接使用`Chatgpt`，但当开了`OpenVPN`到香港后，由于`Chatgpt`的IP限制，反而不是能使用`Chatgpt`了，所以使用`deepseek`的时间更多了，之前跟人吐槽了这个事，他们顺口说了句增加个配置就能解决这个问题。既然有方法能解决这个问题，于是就研究了下，终于搞定了这个事

### 1、使用命令行查找到`Chatgpt`使用的ip
```
nslookup chatgpt.com
```
结果如下：
> nslookup chatgpt.com
>Server:		8.8.8.8
>Address:	8.8.8.8#53
>
>Non-authoritative answer:
Name:	chatgpt.com
Address: 104.18.32.47
Name:	chatgpt.com
Address: 172.64.155.209

### 2、修改`OpenVPN`的配置文件，`xxx.ovpn`
2、找到`OpenVPN`的配置文件，`xxx.ovpn`，这本质上是一个txt文件，使用能打开txt的编辑软件打开, 在之前的route规则下，如果之前没有就在verb3下面增加找到的ip`route 104.18.32.47 255.255.255.255 net_gateway`，打开后的效果如下：
```
client
dev tun
proto udp
remote vpn.example.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client.crt
key client.key
remote-cert-tls server
cipher AES-256-CBC
verb 3

# 针对chatgpt添加的路由规则
route 104.18.32.47 255.255.255.255 net_gateway
route 172.64.155.209 255.255.255.255 net_gateway
<ca>
-----BEGIN CERTIFICATE-----
xxxx
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
xxxx
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
xxxx
-----END PRIVATE KEY-----
</key>
<tls-crypt>
-----BEGIN OpenVPN Static key V1-----
xxxx
-----END OpenVPN Static key V1-----
</tls-crypt>
```

### 3、OpenVPN重新导入配置文件
重新导入修改过后的`xxx.ovpn`文件，开启VPN也可以正常使用`chatgpt`了