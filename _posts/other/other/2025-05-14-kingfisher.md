---
title: kingfiher加载动画及本地文件
author: 独孤流
date: 2025-05-14 10:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

> ### 前言
> 由于项目中使用了多个图片加载框架，有`SDWebImage`、`Kingfisher`\
>同时由于有性能相关需求，需要对gif的加载做优化，同事还推荐了`Gifu`这个框架

为了方便，将项目里全都统一为了`Kingfisher`，看了下`Gifu`的实现逻辑，跟`Kingfisher`是一样的


##### 一、如果只是为了简单播放动画或一般网络图片

如果是一般的网络图片，直接使用`UIImageView`进行加载即可,`Kingfisher`会自动判断文件类型，会返回`animatedImage(with: xxx, duration: xx)`初始化的`UIImage`,这个会自动加载并播放

`retrieveImage` -> `KingfisherManager.provideImage` -> ... -> `KingfisherWrapper<UIImage>.image(data:options:)` 

-> `.GIF` ->  `KingfisherWrapper.animatedImage(data: data, options: options)` → `CGImageSourceCreateWithData` → `options.preloadAll || options.onlyFirstFrame` 

→ `.animatedImage(with: animatedImage.images, duration: duration)`

```
import UIKit
import Kingfisher

let imageView = UIImageView()
let resource = Kingfisher.ImageResource(downloadURL: URL(string: "https://xx.xx/xx"), cacheKey: "https://xx.xx/xx")
// 之所以能自动播放gif图片，是因为Kingfisher内部确定图片类型为gif类型时，通过.animatedImage(with: animatedImage.images, duration: duration)进行初始化，这个就可以自动播放
imageView.kf.setImage(
            with: resource,
            placeholder: placeholder,
            options: nil,
            progressBlock: nil
        )
```



##### 二、如果要精确控制播放次数及回调
如果要监听动态每次循环播放完成、或者指定次数播放完成，需要使用`Kingfisher`里的`Kingfisher.AnimatedImageView`类，这个类可以设置动画循环的次数，且有单次播放完成和全部完成的回调

```
import UIKit
import Kingfisher

class TestGifView: UIView, AnimatedImageViewDelegate {

    // icon
    lazy var avatarView: AnimatedImageView  = {
        let view = AnimatedImageView(frame: .zero)
        view.repeatCount = .finite(count: 3)
        view.delegate = self
        return view
    }()
    // 如果设置了确定的循环次数，全部循环完会回调
    func animatedImageViewDidFinishAnimating(_ imageView: AnimatedImageView) {
        print("finish")
    }
    // 每次循环一遍的最后一帧播放后就会回调
    func animatedImageView(_ imageView: AnimatedImageView, didPlayAnimationLoops count: UInt) {
        print("count: \(count)")
    }
    func showDemo() {
        let resource = Kingfisher.ImageResource(downloadURL: URL(string: "https://xx.xx/xx"), cacheKey: "https://xx.xx/xx")
        avatarView.kf.setImage(
            with: resource,
            placeholder: placeholder,
            options: nil,
            progressBlock: nil
        )
    }
}
```

----
##### 三、加载沙盒里的文件进行播放
一些特殊场景下，比如下载一个资源包进行自己管理也解压，这个时候文件就在本地沙盒里了，针对本地沙盒的文件加载，也可以直接使用`Kingfisher`进行管理,目的是为了自动加载、内存缓存，但是不从默认的硬盘缓存读取，也不是直接从网络下载

```
import UIKit
import Kingfisher

let imageView = UIImageView()
let localURL =  URL(string: "/Users/xxx.xx/xx")
let resource = Kingfisher.ImageResource(downloadURL: localURL, cacheKey: localURL.absoluteString)
let options: KingfisherOptionsInfo = [
    KingfisherOptionsInfoItem.cacheOriginalImage,        // 保证 memory cache 生效（不存磁盘没关系）
    .cacheMemoryOnly,           // 只用内存缓存      // 不限制一定来自缓存（你希望手动控制加载）
    .diskCacheExpiration(.expired), // 明确禁用磁盘缓存
]
imageView.kf.setImage(
    with: resource,
    placeholder: placeholder,
    options: options,
    progressBlock: nil
)
```

---
#### 4、针对下载图片错误处理，比如下载后发现域名被封，替换域名重新下载等场景
```
import UIKit
import Kingfisher

let imageView = UIImageView()
let resource = Kingfisher.ImageResource(downloadURL: URL(string: "https://xx.xx/xx"), cacheKey: "https://xx.xx/xx")
imageView.kf.setImage(
            with: resource,
            placeholder: placeholder,
            options: nil,
            progressBlock: nil
        ) { [weak self] result in
            switch result {
            case .success(let value):
                break
            case .failure(let error):
                // 或者具体分类处理
                switch error {
                case .requestError(let reason):
                    break
                case .responseError(let reason):
                    switch reason {
                        case .invalidURLResponse(let response):
                        break
                        case .invalidHTTPStatusCode(let response):
                        break
                        case .URLSessionError(let responseError):
                        break
                        case .dataModifyingFailed(let task):
                        break
                        case .noURLResponse(let task):
                        break
                        }
                    break
                case .cacheError(let reason):
                    break
                case .processorError(let reason):
                    break
                case .imageSettingError(let reason):
                    break
                }
            }
        }
```
----

##### 5、Kingfisher里的读取图片类型的代码,是读取前8个字节进行文件类型判断
```
import Foundation

/// Represents image format.
///
/// - unknown: The format cannot be recognized or not supported yet.
/// - PNG: PNG image format.
/// - JPEG: JPEG image format.
/// - GIF: GIF image format.
public enum ImageFormat {
    /// The format cannot be recognized or not supported yet.
    case unknown
    /// PNG image format.
    case PNG
    /// JPEG image format.
    case JPEG
    /// GIF image format.
    case GIF
    
    struct HeaderData {
        static var PNG: [UInt8] = [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]
        static var JPEG_SOI: [UInt8] = [0xFF, 0xD8]
        static var JPEG_IF: [UInt8] = [0xFF]
        static var GIF: [UInt8] = [0x47, 0x49, 0x46]
    }
    
    /// https://en.wikipedia.org/wiki/JPEG
    public enum JPEGMarker {
        case SOF0           //baseline
        case SOF2           //progressive
        case DHT            //Huffman Table
        case DQT            //Quantization Table
        case DRI            //Restart Interval
        case SOS            //Start Of Scan
        case RSTn(UInt8)    //Restart
        case APPn           //Application-specific
        case COM            //Comment
        case EOI            //End Of Image
        
        var bytes: [UInt8] {
            switch self {
            case .SOF0:         return [0xFF, 0xC0]
            case .SOF2:         return [0xFF, 0xC2]
            case .DHT:          return [0xFF, 0xC4]
            case .DQT:          return [0xFF, 0xDB]
            case .DRI:          return [0xFF, 0xDD]
            case .SOS:          return [0xFF, 0xDA]
            case .RSTn(let n):  return [0xFF, 0xD0 + n]
            case .APPn:         return [0xFF, 0xE0]
            case .COM:          return [0xFF, 0xFE]
            case .EOI:          return [0xFF, 0xD9]
            }
        }
    }
}


extension Data: KingfisherCompatibleValue {}

// MARK: - Misc Helpers
extension KingfisherWrapper where Base == Data {
    /// Gets the image format corresponding to the data.
    public var imageFormat: ImageFormat {
        guard base.count > 8 else { return .unknown }
        
        var buffer = [UInt8](repeating: 0, count: 8)
        base.copyBytes(to: &buffer, count: 8)
        
        if buffer == ImageFormat.HeaderData.PNG {
            return .PNG
            
        } else if buffer[0] == ImageFormat.HeaderData.JPEG_SOI[0],
            buffer[1] == ImageFormat.HeaderData.JPEG_SOI[1],
            buffer[2] == ImageFormat.HeaderData.JPEG_IF[0]
        {
            return .JPEG
            
        } else if buffer[0] == ImageFormat.HeaderData.GIF[0],
            buffer[1] == ImageFormat.HeaderData.GIF[1],
            buffer[2] == ImageFormat.HeaderData.GIF[2]
        {
            return .GIF
        }
        
        return .unknown
    }
    
    public func contains(jpeg marker: ImageFormat.JPEGMarker) -> Bool {
        guard imageFormat == .JPEG else {
            return false
        }
        
        var buffer = [UInt8](repeating: 0, count: base.count)
        base.copyBytes(to: &buffer, count: base.count)
        for (index, item) in buffer.enumerated() {
            guard
                item == marker.bytes.first,
                buffer.count > index + 1,
                buffer[index + 1] == marker.bytes[1] else {
                continue
            }
            return true
        }
        return false
    }
}

```
----

##### 6、Kingfisher支持webp
Podfile
```
pod 'Kingfisher'
pod 'KingfisherWebP'

# 如果M系列芯片要按x86编译
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['EXCLUDED_ARCHS[sdk=iphonesimulator*]'] = 'arm64'
    end
  end
end
```

在启动的地方配置
```
KingfisherManager.shared.defaultOptions += [
  .processor(WebPProcessor.default),
  .cacheSerializer(WebPSerializer.default)
]
```

在使用的地方正常使用即可