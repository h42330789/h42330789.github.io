---
title: 使用UITextView展示富文本及图片点击
author: 独孤流
date: 2025-05-23 01:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

> ### 前言
> 项目里有加载富文本的场景，而且富文本里的图片在点击时还要支持方法，这就需要识别图片的点击，且要识别点击的是哪个图片，最好和已有的图片缓存框架集合在一起

主要实现思路是将文本里的图片，使用正则找出图片地址`https://xxx/xx.png`，然后使用本地`file://xxx/xx.png`的方式替换，然后名字变成md5后，然后通过md5反向找到文件url，如果图片已经下载好了就使用真实图片，图片没有下载好久使用占位图，实现demo如下：

```
//
//  ViewController.swift
//  TestAttr
//
//  Created by MacBook Pro on 5/22/25.
//

import UIKit

class ViewController: UIViewController, UITextViewDelegate {
    
    lazy var textView1 = UITextView(frame: CGRect(x: 100, y: 100, width: 200, height: 200))
    lazy var textView2 = UITextView(frame: CGRect(x: 100, y: 350, width: 200, height: 200))
    var md5Dict: [String: String] = [:]
    var urlList: [String] = []
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.backgroundColor = .white
        textView1.backgroundColor = .orange
        textView1.delegate = self
        textView1.isEditable = false
        textView2.backgroundColor = .green
        textView2.delegate = self
        textView2.isEditable = false
        let text = """
<html>
<head>
    <style>
    img {width: 50px; height: 50px;}
    </style>
</head>
<body>
神舟二十号航天
<img src=\"https://cdn.pixabay.com/photo/2021/10/14/12/32/autumn-6708984_1280.jpg\" />
<img src=\"https://cdn.pixabay.com/photo/2024/05/14/11/39/tv-8760958_1280.png\" />
中美各取消91%关税
<img src=\"https://cdn.pixabay.com/photo/2025/05/10/15/40/bear-9591466_1280.jpg\" />
</body>
</html>
"""
        self.view.addSubview(textView1)
        self.view.addSubview(textView2)
        
        textView1.attributedText = text.htmlToAttr
        let (attr, urlList, urlDict) = text.htmlToAttrCustomImage(placeholder: UIImage(named: "default_cover.jpg")) { [weak self]_ in
            DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(10), execute: {
                self?.loadHtmlAttr(text: text)
            })
        }
        textView2.attributedText = attr
        self.md5Dict = urlDict
        self.urlList = urlList
        
    }
    func loadHtmlAttr(text: String) {
        let (attr, urlList, urlDict) = text.htmlToAttrCustomImage(placeholder: UIImage(named: "default_cover.jpg"))
        textView2.attributedText = attr
    }

    func textView(_ textView: UITextView,
                  shouldInteractWith textAttachment: NSTextAttachment,
                  in characterRange: NSRange,
                  interaction: UITextItemInteraction) -> Bool {
        // 可以执行自定义逻辑，比如打开图片、弹窗等
        // 示例：通知回调或打开大图预览
        var findFileName = ""
        if let fileName = textAttachment.fileWrapper?.filename {
            findFileName = fileName
        } else if let fileName = textAttachment.fileWrapper?.preferredFilename {
            findFileName = fileName
        }
        if findFileName.count > 0 {
            let fileMd5 = findFileName.components(separatedBy: ".").first ?? ""
            let fileUrl = self.md5Dict[fileMd5] ?? ""
            print("findFileName: \(findFileName)")
            print("fileMd5: \(fileMd5)")
            print("fileUrl: \(fileUrl)")
        }
        

        return false // false 表示你已经处理，不交给系统默认行为
    }


}

import CommonCrypto
extension String {
    func md5() -> String {
        let data = Data(self.utf8)
        var digest = [UInt8](repeating: 0, count: Int(CC_MD5_DIGEST_LENGTH))

        data.withUnsafeBytes {
            _ = CC_MD5($0.baseAddress, CC_LONG(data.count), &digest)
        }

        return digest.map { String(format: "%02x", $0) }.joined()
    }
    var htmlToAttr: NSAttributedString? {
        guard let data = self.data(using: .unicode) else {
           return nil
       }
       let options = [
           NSAttributedString.DocumentReadingOptionKey.documentType: NSAttributedString.DocumentType.html
       ]
        return try? NSAttributedString(data: data, options: options, documentAttributes: nil)
    }

    /// 富文本渲染，图片已经存在则以本地路径file://的方式处理，不存在则以占位图写入沙盒，然后下载图片
    func htmlToAttrCustomImage(placeholder: UIImage? = nil, uniqueId: String? = nil, handler: ((String?) -> Void)? = nil) -> (NSAttributedString?, [String], [String: String]) {
        let pattern = #"<\s*img\s+[^>]*src\s*=\s*['\"](.*?)['\"][^>]*?>"#
        guard let regex = try? NSRegularExpression(pattern: pattern, options: [.caseInsensitive]) else {
            return (nil, [], [:])
        }
        
        let nsStr = self as NSString
        let matches = regex.matches(in: self, options: [], range: NSRange(location: 0, length: nsStr.length))
        
        var urlToMd5: [String: String] = [:]
        var processed = self
        let placeholderImage = placeholder ?? UIImage()
        
        var sources: [String] = []
        for match in matches.reversed() {
            guard match.numberOfRanges >= 2 else { continue }
            
            // 1. 获取原始 src 值
            let srcRange = match.range(at: 1)
            var originUrlMd5 = ""
            if let swiftSrcRange = Range(srcRange, in: processed) {
                let src = String(processed[swiftSrcRange])
                originUrlMd5 = src.md5()
                urlToMd5[originUrlMd5] = src
                sources.insert(src, at: 0) // 保持顺序
            }
            let originUrl = urlToMd5[originUrlMd5] ?? ""
            // 缓存和下载功能，要在实际项目中使用sdwebimage或KingFisher这样的成熟框架，本demo只是为了不依赖任何库做的简化操作
            // 2. 替换整个 <img> 标签
            // 缓存查找图片是否存在
            // 拼接保存路径
            var fileURL: URL? = String.fileUrl(dir: "reals", name: "\(originUrlMd5).\(originUrl.components(separatedBy: ".").last ?? "")")
            if let path = fileURL?.path, FileManager.default.fileExists(atPath: path) == false {
                // 生成图片+下载图片
                fileURL = String.saveImageToDocuments(image: placeholderImage, fileName: "\(originUrlMd5).jpg", dir: "placeholders")
                DispatchQueue.global().async {
                    String.downloadImg(urlString: originUrl, uniqueId: uniqueId, completion: handler)
                }
            } else {
                // 图片已存在
                print("图片已存在 \(originUrlMd5) \(originUrl)")
            }
            if let url = fileURL, FileManager.default.fileExists(atPath: url.path) {
                let replacement = "<img src=\"\(url)\">"
                let fullRange = match.range
                if let swiftFullRange = Range(fullRange, in: processed) {
                    processed.replaceSubrange(swiftFullRange, with: replacement)
                }
            }
        }
        guard let attr = processed.htmlToAttr else {
            return (nil, sources, urlToMd5)
        }
        return (attr, sources, urlToMd5)
        
    }
    static func fileUrl(dir: String, name: String) -> URL {
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let dirPath = documentsURL.appendingPathComponent("\(dir)").path()
        if FileManager.default.fileExists(atPath: dirPath) == false {
            do {
                try FileManager.default.createDirectory(atPath: dirPath, withIntermediateDirectories: true, attributes: nil)
            } catch {
                
            }
        }
        // 拼接保存路径
        return documentsURL.appendingPathComponent("\(dir)/\(name)")
    }
    static func saveImageToDocuments(image: UIImage, fileName: String, dir: String) -> URL? {
        var fileData: Data? = nil
        if let data = image.pngData() {
            fileData = data
        } else if let data = image.jpegData(compressionQuality: 1.0) {
            fileData = data
        }
        guard let data = fileData else {
            return nil
        }

        // 获取沙盒 Documents 目录路径
        // 拼接保存路径
        let fileURL = String.fileUrl(dir: dir, name: fileName)
        if FileManager.default.fileExists(atPath: fileURL.path) {
            return fileURL
        }
        do {
            // 写入文件
            try data.write(to: fileURL)
            return fileURL
        } catch {
            return nil
        }
    }
    
    static func downloadImg(urlString: String, uniqueId: String?, completion: ((String?) -> Void)? = nil) {
        guard let url = URL(string: urlString) else {
               completion?(uniqueId)
               return
           }

           URLSession.shared.dataTask(with: url) { data, response, error in
               guard let data = data, error == nil, let image = UIImage(data: data) else {
                   print("图片下载失败：\(error?.localizedDescription ?? "未知错误")")
                   completion?(uniqueId)
                   return
               }
               let originUrlMd5 = urlString.md5()
               let _ = String.saveImageToDocuments(image: image, fileName: "\(originUrlMd5).\(urlString.components(separatedBy: ".").last ?? "")", dir: "reals")
               let placeholderURL = String.fileUrl(dir: "placeholders", name: "\(originUrlMd5).jpg")
               if FileManager.default.fileExists(atPath: placeholderURL.path) {
                   do {
                       try FileManager.default.removeItem(at: placeholderURL)
                   } catch {
                       
                   }
               }
               completion?(uniqueId)
           }.resume()
    }
}

```

与sdwebimage结合的例子
```
import CommonCrypto
import SDWebImage
extension String {
    func md5() -> String {
        let data = Data(self.utf8)
        var digest = [UInt8](repeating: 0, count: Int(CC_MD5_DIGEST_LENGTH))

        data.withUnsafeBytes {
            _ = CC_MD5($0.baseAddress, CC_LONG(data.count), &digest)
        }

        return digest.map { String(format: "%02x", $0) }.joined()
    }
    var htmlToAttr: NSAttributedString? {
        guard let data = self.data(using: .unicode) else {
           return nil
       }
       let options = [
           NSAttributedString.DocumentReadingOptionKey.documentType: NSAttributedString.DocumentType.html
       ]
        return try? NSAttributedString(data: data, options: options, documentAttributes: nil)
    }
 var sdWebimageStorName: String {
        let key = self.md5()
        var ext = self.components(separatedBy: ".").last ?? "png"
        if self.hasPrefix("data:image/") {
            ext = "png"
        }
        let name = "\(key).\(ext)"
        return name
    }
    var sdWebimageStorUrl: String {
        let key = self.sdWebimageStorName
        let path = SDImageCache.shared.cachePath(forKey: key)
        return path ?? ""
    }
    
    /// 富文本渲染，图片已经存在则以本地路径file://的方式处理，不存在则以占位图写入沙盒，然后下载图片
    func htmlToAttrCustomImage(placeholder: UIImage? = nil, uniqueId: String? = nil, handler: ((String?) -> Void)? = nil) -> (NSAttributedString?, [String], [String: String]) {
        let pattern = #"<\s*img\s+[^>]*src\s*=\s*['\"](.*?)['\"][^>]*?>"#
        guard let regex = try? NSRegularExpression(pattern: pattern, options: [.caseInsensitive]) else {
            return (nil, [], [:])
        }
        
        let nsStr = self as NSString
        let matches = regex.matches(in: self, options: [], range: NSRange(location: 0, length: nsStr.length))
        
        var urlToMd5: [String: String] = [:]
        var processed = self
        let placeholderImage = placeholder ?? UIImage()
        
        var sources: [String] = []
        for match in matches.reversed() {
            guard match.numberOfRanges >= 2 else { continue }
            
            // 1. 获取原始 src 值
            let srcRange = match.range(at: 1)
            var originUrlMd5 = ""
            if let swiftSrcRange = Range(srcRange, in: processed) {
                let src = String(processed[swiftSrcRange])
                // sdwebimage真实的文件名为key.md5
                originUrlMd5 = src.sdWebimageStorName.md5()
                urlToMd5[originUrlMd5] = src
                sources.insert(src, at: 0) // 保持顺序
            }
            let originUrl = urlToMd5[originUrlMd5] ?? ""
            // 缓存和下载功能，要在实际项目中使用sdwebimage或KingFisher这样的成熟框架，本demo只是为了不依赖任何库做的简化操作
            // 2. 替换整个 <img> 标签
            // 缓存查找图片是否存在
            // 拼接保存路径
            var filePath = originUrl.sdWebimageStorUrl
            
            if SDImageCache.shared.diskCache.containsData(forKey: originUrl.sdWebimageStorName) == false {
                // 如果是base64图片，直接转换为图片存储
                if originUrl.hasPrefix("data:image/"),
                    let dataStr = originUrl.components(separatedBy: "base64,").last,
                   let data = Data(base64Encoded: dataStr, options: .ignoreUnknownCharacters) {
                    SDImageCache.shared.storeImageData(data, forKey: originUrl.sdWebimageStorName)
                } else {
                    // 生成图片+下载图片
                    filePath = SPPathKit.saveImageAndFIlePath(dir: "placeholders", fileName: "\(originUrlMd5).png",  image: placeholderImage)
                    if originUrl.lowercased().starts(with: "http://") || originUrl.lowercased().starts(with: "https://") {
                        // 网络图片才使用SDImageCache下载
                        DispatchQueue.global().async {
                            SDWebImageDownloader.shared.downloadImage(with: URL(string: originUrl)) { image, data, error, finished in
                                if finished, let data = data {
                                    // 保存下载的图片
                                    SDImageCache.shared.storeImageData(toDisk: data, forKey: originUrl.sdWebimageStorName)
                                    // 删除展位图
                                    let placeholderPath = SPPathKit.createdirAndFilePath(dir: "placeholders", fileName: "\(originUrlMd5).png")
                                    SPPathKit.remove(placeholderPath)
                                    // 回调
                                    handler?(uniqueId)
                                }
                            }
                        }
                    }
                }
                
            }
            if FileManager.default.fileExists(atPath: filePath) {
                let replacement = "<img src=\"file://\(filePath)\">"
                let fullRange = match.range
                if let swiftFullRange = Range(fullRange, in: processed) {
                    processed.replaceSubrange(swiftFullRange, with: replacement)
                }
            }
        }
        guard let attr = processed.htmlToAttr else {
            return (nil, sources, urlToMd5)
        }
        return (attr, sources, urlToMd5)
    }
}
```