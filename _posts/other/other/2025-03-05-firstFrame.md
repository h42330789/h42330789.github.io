---
title: swift远端视频第一帧方法及问题
author: 独孤流
date: 2025-03-05 10:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

> ### 前言
在往常的业务开发中，对于视频的交互展示，一般都是有一个专门的封面图片地址，一个专门的视频地址两个字段组成，这样做的原因：
1、在列表或默认展示这个封面图片，点击封面图片或播放按钮才正式播放视频，这样可以在不播放时能不用下载视频，节省服务器流量及用户的手机流量。\
2、另一个常用场景就是在聊天列表里的视频，由于通常视频和图片都是加密的，没法直接从加密视频里获取第一帧图片，所以需要一个专门的封面图片，否则必须要把全部视频下载完成同时解密后获取第一帧作为封面

#### 业务中的新通点
1、开发过程中要管理图片地址和视频地址两个字段比较繁琐\
2、由于一些没有加密的视频，标准的oss存储视频一般提供直接拼接地址获取到视频封面\
3、对于一些没有加密的视频，开发时间不够或其他特殊的一些人不配合，希望展示方自己想办法从视频里获取第一帧图


----
### 1、如果视频没有加密

方案一、老老实实使用封面图地址 + 视频地址 的方式实现

方案二、 视频没有加密，视频又是存储在支持通过配置链接直接从oss服务器拉去的，直接通过配置链接的方式获取，具体参考各个oss服务的文档,以阿里云oss为例
```
假设你的视频 URL 是：
https://your-bucket-name.oss-cn-hangzhou.aliyuncs.com/your-video-file.mp4

在 URL 后拼接以下参数：
?x-oss-process=video/snapshot,t_0,f_jpg,w_0,h_0,m_fast

完整 URL：
https://your-bucket-name.oss-cn-hangzhou.aliyuncs.com/your-video-file.mp4?x-oss-process=video/snapshot,t_0,f_jpg,w_0,h_0,m_fast
```

方案三、 视频没有加密，但是视频是在不支持的自己服务器上，或者不太确定是否支持通过配置链接获取视频第一帧，直接使用代码方式获取视频第一帧，方法如下：
```
typealias ActionBlock = (()->Void)
func dispatchGlobalQueue(action: @escaping ActionBlock) -> Void {
    if Thread.isMainThread {
        DispatchQueue.global().async {
            action()
        }
        return
    }
    action()
}
func dispatchMainQueue(action: @escaping ActionBlock) -> Void {
    
    if Thread.isMainThread {
        action()
        return
    }
    DispatchQueue.main.async {
        action()
    }
}
extension String {
    func syncRetrieveImage() -> UIImage? {
        let key = self
        var retrievedImage: UIImage? = nil
        let semaphore = DispatchSemaphore(value: 0)  // 创建一个信号量
        
        // 异步调用，使用信号量同步等待
        KingfisherManager.shared.cache.retrieveImage(forKey: key) { result in
            switch result {
            case .success(let value):
                retrievedImage = value.image
            case .failure:
                retrievedImage = nil
            }
            
            semaphore.signal()  // 信号量发信号，允许线程继续
        }
        
        // 等待信号量完成
        semaphore.wait()
        
        return retrievedImage
    }
    typealias VideoFirstFrameBlock = (UIImage?) -> Void
    func fetchVideoFirstFrame(completion: VideoFirstFrameBlock? = nil) {
        dispatchGlobalQueue {
            self.realFetchVideoFirstFrame(completion: completion)
        }
    }
    private func realFetchVideoFirstFrame(delayTimes: [Double] = [0, 1, 5, 10], completion: VideoFirstFrameBlock? = nil) {
        let key = self
        guard let url = URL(string: key) else {
            dispatchMainQueue {
                completion?(nil)
            }
            return
        }
        // 如果存在已经缓存过的封面，直接使用缓存数据，不再继续下载
        if let image = key.syncRetrieveImage() {
            dispatchMainQueue {
                completion?(image)
            }
            return
        }
        
        let asset = AVURLAsset(url: url)
        let generator = AVAssetImageGenerator(asset: asset)
        generator.appliesPreferredTrackTransform = true // 保证视频方向正确
        
        // 精确获取时间点，减少处理范围
        generator.requestedTimeToleranceBefore = .zero
        generator.requestedTimeToleranceAfter = .zero
        let secondFrom = delayTimes.first ?? 0
        let time = CMTime(seconds: secondFrom, preferredTimescale: 600) // 0秒处
        
        generator.generateCGImagesAsynchronously(forTimes: [NSValue(time: time)]) { _, cgImage, _, result, error in
            switch result {
            case .succeeded:
                guard let cgImage = cgImage else {
                    DispatchQueue.main.async { completion?(nil) }
                    return
                }
                // 如果下载到了视频封面，缓存起来
                let image = UIImage(cgImage: cgImage)
                KingfisherManager.shared.cache.store(image, forKey: key)
                dispatchMainQueue {
                    completion?(image)
                }
            case .failed:
                if delayTimes.count > 1 {
                    print("Error generating secondFrom:\(secondFrom) age: \(error?.localizedDescription ?? "Unknown error") \(url)")
                    let nextTimes = delayTimes.subArray(from: 1, size: delayTimes.count - 1)
                    key.realFetchVideoFirstFrame(delayTimes: nextTimes, completion: completion)
                } else {
                    dispatchMainQueue {
                        completion?(nil)
                    }
                }
            case .cancelled:
                print("Error generating image: \(error?.localizedDescription ?? "Unknown error") \(url)")
                dispatchMainQueue {
                    completion?(nil)
                }
            @unknown default:
                dispatchMainQueue {
                    completion?(nil)
                }
            }
        }
    }
}

// 使用方式
let serverUrl = "https://xxx.xxx.xx/xx/xx.mp4"
serverUrl.fetchVideoFirstFrame()
```
该方案遇到的一些问题：一般的视频都可以，上传的一些比较短的视频，第1秒是黑的，其实是没有，这个把视频下载到Mac上也能看出来，遇到这种错误，就做了策略继续往后取1，在没有就取5、10这样递归取，都没有就不管了

方案四、将视频完整下载下来，然后再获取下载下来的视频的第一帧（这种方式不推荐，浪费流量，没有播放也把完整视频下载下来了）

代码同方案三，只是传递的地址不一样
```
// 1 ....下载视频，或者APP里本身包含的视频
let localPath = "file://xx/xx/xx"
localPath.fetchVideoFirstFrame()
```
----
### 2、如果视频加密了
方案一 老老实实使用封面图地址 + 视频地址 两个字段处理

方案二 必须自己把视频全部下载下来，然后解密，解密后再获取视频的第一帧
