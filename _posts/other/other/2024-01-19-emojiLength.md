---
title: emoji计算长度
author: 独孤流
date: 2024-01-19 01:04:00 +0800
categories: [other, 其他]
tags: [emoji]     # TAG names should always be lowercase
---

在处理包含emoji的代码进行截取时，遇到了崩溃，调查原因是计算截取文字位置时，`Objective-C`里的`NSString`与`swift`里的`String`不一致\
以下为同事研究总结的内容
>跟大家分享一点关于字符串长度的知识点
 1. OC中的NSString.length与Swift中String.count并不一定相等
 2. NSString.length返回的是字符串utf16编码的长度, String.count返回的是unicode码的个数
 3. 因此 String.utf16.count才等于NSString.length
 4. textview.selectedRange.location获取到的光标位置, 是以utf16编码的长度为计数单位的, 因此设计到取光标位置的时候, 字符串的操作都应该用长度来计数, 而非String.count
 5. 一个emoji表情的length通常是2, 组合型emoji如👨‍👩‍👧‍👧可能超过10, 但是体现在String.count里都是1

----
### 案例一：`UITextView`获取`selectedRange.location`里，按range截取崩溃的问题
```
extension String {
     subscript (range: Range<Int>) -> String {
        get {
            var fromOffset = range.lowerBound
            if range.lowerBound > self.count {
                fromOffset = self.count
            }
            var toOffset = range.upperBound
            if range.upperBound > self.count {
                toOffset = self.count
            }
            let startIndex = self.index(self.startIndex, offsetBy: fromOffset)
            let endIndex = self.index(self.startIndex, offsetBy: toOffset)
            return String(self[startIndex..<endIndex])
        } set {
            let startIndex = self.index(self.startIndex, offsetBy: range.lowerBound)
            let endIndex = self.index(self.startIndex, offsetBy: range.upperBound)
            let strRange = startIndex..<endIndex
            self.replaceSubrange(strRange, with: newValue)
        }
    }
}
textView.text // "did😗🙂"
// swift里String的count、OC里NSString的length的区别
textView.text.length // 5
textView.text.count // 5
(textView.text as NSString).length // 7
// 获取textView里的range
textView.selectedRange // {location:7, length:0} ,这里的7就是因为把emoji按2个长度计算了
let cursorLoc = textView.selectedRange.location // 7
let frontText = textView.text[0..<cursorLoc] // ❌❌❌会崩溃，因为截取的字符串长度比原始字符串还长❌❌❌
let frontText2 = (textView.text as NSString).substring(with: NSMakeRange(0, cursorLoc))
```

### 案例二：使用`NSMutableAttributedString`设置属性时，swift里`String`的`range(of: xxx)`方法与oc里`NSString`的`range(of: xxx)`方法结果不一样
```
let text = "did😗🙂"
let ocText = (text as NSString)
if let findRange = text.range(of: "did😗") {
   let start = text.distance(from: text.startIndex, to: findRange.lowerBound)
   let length = text.distance(from: findRange.lowerBound, to: findRange.upperBound)
   let swiftRange = NSMakeRange(start, length) // {0, 4}
   let attStr = NSMutableAttributedString(string: text, attributes: [NSAttributedString.Key.foregroundColor: UIColor.black])
   attStr.addAttribute(.foregroundColor, value: UIColor.red, range: swiftRange)
   
   let label = UILabel()
   label.attributedText = attStr
   label.frame = CGRect(x: 100, y: 300, width: 220, height: 40)
   self.view.addSubview(label)
 }

 let ocRange = ocText.range(of: "did😗") // {0, 5}
   let label2 = UILabel()
   let attStr2 = NSMutableAttributedString(string: text, attributes: [NSAttributedString.Key.foregroundColor: UIColor.black])
   attStr2.addAttribute(.foregroundColor, value: UIColor.red, range: ocRange)
   label2.attributedText = attStr2
   label2.frame = CGRect(x: 100, y: 350, width: 220, height: 40)
   self.view.addSubview(label2)
```
![image](/assets/img/other/emoji_range0.png)
![image](/assets/img/other/emoji_range1.png)

----
遍历查询所有符合条件的range
```
func test() {
    var startIndex: Int = 0
    let text = "did😗🙂did😗addfdid😗"
    var ocText: NSString = text as NSString
    var findStr1 = "did😗"
    var findRange = self.findAllRange(ocText: ocText, startIndex: startIndex, findStr1: findStr1)
    while findRange.length > 0 {
        startIndex = findRange.location + findRange.length
        findRange = self.findAllRange(ocText: ocText, startIndex: startIndex, findStr1: findStr1)
        if findRange.length <= 0 {
            break
        }
    }
    let rangeList = text.findAllRange(findStr: findStr1)
}
func findAllRange(ocText: NSString, startIndex: Int, findStr1: String) -> NSRange {
     if startIndex >= ocText.length {
         return NSRange(location: 0, length: 0)
     }
     let subLen = ocText.length - startIndex
     if subLen <= 0 {
         return NSRange(location: 0, length: 0)
     }
     // location: 包含要截取的第一个， length: 从location开始要截取的长度
     let ocSubStr = (startIndex == 0) ? ocText : (ocText.substring(with: NSRange(location: startIndex, length: subLen)) as NSString) 

     var findRange = ocSubStr.range(of: findStr1)
     if findRange.length <= 0 {
         return NSRange(location: 0, length: 0)
     }
     let resultRange = NSRange(location: startIndex+findRange.location, length: findRange.length)
     return resultRange
 }

使用分类的方式实现
extension String {
    func findAllRange(findStr: String) -> [NSRange] {
         var ocText: NSString = (self as NSString)
         
        var rangeList: [NSRange] = []
        var startIndex: Int = 0
        // 查找第一遍
        var findRange = self.findStrRange(ocText: ocText, startIndex: startIndex, findStr1: findStr1)
        
        while findRange.length > 0 {
            // 如果找到了，继续遍历查找
            rangeList.append(findRange)
            startIndex = findRange.location + findRange.length
            findRange = self.findStrRange(ocText: ocText, startIndex: startIndex, findStr1: findStr1)
            if findRange.length <= 0 {
                // 查找结束后，结束遍历
                break
            }
        }
        return rangeList
     }
    func findStrRange(ocText: NSString, startIndex: Int, findStr1: String) -> NSRange {
         if startIndex >= ocText.length {
             return NSRange(location: 0, length: 0)
         }
         let subLen = ocText.length - startIndex
         if subLen <= 0 {
             return NSRange(location: 0, length: 0)
         }
         // location: 包含要截取的第一个， length: 从location开始要截取的长度
         let ocSubStr = (startIndex == 0) ? ocText : (ocText.substring(with: NSRange(location: startIndex, length: subLen)) as NSString)

         var findRange = ocSubStr.range(of: findStr1)
         if findRange.length <= 0 {
             return NSRange(location: 0, length: 0)
         }
         let resultRange = NSRange(location: startIndex+findRange.location, length: findRange.length)
         return resultRange
     }
}