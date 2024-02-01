---
title: 安装GoogleCloud
author: 独孤流
date: 2024-01-27 01:04:00 +0800
categories: [other, 其他]
tags: [googleCloud]     # TAG names should always be lowercase
---

近期在搞itms服务器时，iOS17以后得账号使用http的地方放ipa包会安装不上，于是直接将ipa包放到google cloud上，于是需要配置googleCloud

文档地址：https://cloud.google.com/sdk/docs/install-sdk?hl=zh-cn

1、下载安装包，根据电脑是X86还是M系列芯片选择\

![image](/assets/img/other/google_cloud1.png)

```
# 执行安装命名
./google-cloud-sdk/install.sh

# 由于我们使用json key验证 所以不用gcloud init,使用json的方式做验证
./google-cloud-sdk/bin/gcloud auth activate-service-account --key-file=/xxx/xxx/xxx-gcs.json

# 将test.zip 到 google cloud storage , 桶的名称比如bucketNameA
gsutil cp /xx/xx/test.zip gs://bucketNameA//file/test.zip

# 查看google cloud storage 上的文件
gsutil ls -r gs://bucketNameA/

# 刪除google cloud storage 的指定文件
gsutil rm  gs://bucketNameA/file/test.zip

下载地址 https://storage.googleapis.com/bucketNameA/file/test.zip
```