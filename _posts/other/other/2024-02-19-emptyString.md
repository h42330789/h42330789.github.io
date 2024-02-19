---
title: 去掉空白字符及计算宽高
author: 独孤流
date: 2024-02-19 01:04:00 +0800
categories: [other, 其他]
tags: [String]     # TAG names should always be lowercase
---
参考：
- https://copyprogramming.com/howto/value-of-type-string-has-no-member-stringbytrimmingcharactersinset

本文demo的代码
- https://github.com/h42330789/StudyIM/blob/feature/ItemListController/StudyAsynDisplay/StudyAsynDisplay/TestStringVC.swift
- https://github.com/h42330789/StudyIM/blob/feature/ItemListController/StudyAsynDisplay/StudyAsynDisplay/UILabel+size.m
> ### 前言
> 在日常开发中，经常需要根据需求去掉`开始处`或`结尾处`或`开始和结尾处`字符串中的空格，但是大部分资料都是只有去掉全部空白符，特搜集整理下

Objective-C代码
```
@interface NSString (TrimmingAdditions)
typedef NS_ENUM(NSInteger, TrimmingType) {
    leading, // 开头处的空白 " ab c " -> "ab c "
    trailing, // 结尾处的空白 " ab c " -> "ab c "
    leadingAndTrailing, // 开始和结尾处的空白 " ab c " -> "ab c"
    all // 全部的空白 " ab c " -> "abc"
};
- (NSString *)trimWhitespaceWithType:(TrimmingType)type;
- (NSString *)trimWhitespaceAndNewLineWithType:(TrimmingType)type;
@end
@implementation NSString (TrimmingAdditions)

- (NSString *)trimWhitespaceWithType:(TrimmingType)type {
    return [self trimWithType:type characterInset:[NSCharacterSet whitespaceCharacterSet]];
}
- (NSString *)trimWhitespaceAndNewLineWithType:(TrimmingType)type {
    return [self trimWithType:type characterInset:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
}
- (NSString *)trimWithType:(TrimmingType)type characterInset:(NSCharacterSet *)characterSet {
    switch (type) {
        case leading:
            return [self stringByTrimmingLeadingCharactersInSet:characterSet];
        case trailing:
            return [self stringByTrimmingTrailingCharactersInSet:characterSet];
        case leadingAndTrailing:
            return [self stringByTrimmingCharactersInSet:characterSet];
        case all:
            return [self stringByReplacingOccurrencesOfString:@" " withString:@""];
    }
}
- (NSString *)stringByTrimmingLeadingCharactersInSet:(NSCharacterSet *)characterSet {
    NSUInteger location = 0;
    NSUInteger length = [self length];
    unichar charBuffer[length];
    [self getCharacters:charBuffer];

    for (NSInteger i= 0; i < length; i++) {
        if (![characterSet characterIsMember:charBuffer[i]]) {
            location = i;
            break;
        }
    }

    return [self substringWithRange:NSMakeRange(location, length - location)];
}

- (NSString *)stringByTrimmingTrailingCharactersInSet:(NSCharacterSet *)characterSet {
    NSUInteger location = 0;
    NSUInteger length = [self length];
    unichar charBuffer[length];
    [self getCharacters:charBuffer];

    for (NSInteger i = length; i > 0; i--) {
        if (![characterSet characterIsMember:charBuffer[i - 1]]) {
            length = i;
            break;
        }
    }

    return [self substringWithRange:NSMakeRange(location, length - location)];
}

@end
```
测试结果如下：
```
NSString *text = @" ab c ";
// 只去掉空格
NSString *trimLeftText = [text trimWhitespaceWithType:leading];
NSString *trimRightText = [text trimWhitespaceWithType:trailing];
NSString *trimBothText = [text trimWhitespaceWithType:leadingAndTrailing];
NSString *trimAllText = [text trimWhitespaceWithType:all];
NSLog(@"text: %@", text); // " ab c "
NSLog(@"trimLeftText: %@", trimLeftText); // "ab c "
NSLog(@"trimRightText: %@", trimRightText); // " ab c"
NSLog(@"trimBothText: %@", trimBothText); // "ab c"
NSLog(@"trimAllText: %@", trimAllText); // "abc"
```

Swift代码：
```
extension String {
    enum TrimType {
        case leading
        case trailing
        case leadingAndTrailing
        case all
    }
    func trimWhitespace(type: TrimType) -> String {
        return self.trimWhitespace(type: type, characterSet: .whitespaces)
    }
    func trimWhitespaceAndNewline(type: TrimType) -> String {
        return self.trimWhitespace(type: type, characterSet: .whitespacesAndNewlines)
    }
    func trimWhitespace(type: TrimType, characterSet: CharacterSet) -> String {
        switch type {
        case .leading:
            return self.stringByTrimLeading(characterSet: characterSet)
        case .trailing:
            return self.stringByTrimTrailing(characterSet: characterSet)
        case .leadingAndTrailing:
            return self.trimmingCharacters(in: characterSet)
        case .all:
            return self.replacingOccurrences(of: " ", with: "")
        }
    }
    func stringByTrimLeading(characterSet: CharacterSet) -> String {
        var location: Int = 0
        var length: Int = self.count
        for scalar in self.unicodeScalars {
            if characterSet.contains(scalar) == false {
                break
            }
            location = location + 1
        }
        let startIndex = self.index(self.startIndex, offsetBy: location)
        let endIndex = self.index(self.startIndex, offsetBy: length)
        return String(self[startIndex..<endIndex])
    }
    
    func stringByTrimTrailing(characterSet: CharacterSet) -> String {
        var location: Int = 0
        var length: Int = self.count
        for scalar in self.unicodeScalars.reversed() {
            if characterSet.contains(scalar) == false {
                break
            }
            length = length - 1
        }
        let startIndex = self.index(self.startIndex, offsetBy: location)
        let endIndex = self.index(self.startIndex, offsetBy: length)
        return String(self[startIndex..<endIndex])
    }

    func replaceTrailingSpace(withText: String = "1") -> String {
        let list = self.components(separatedBy: "\n")
        var listNew: [String] = []
        for str in list {
            let strNew = str.trimWhitespace(type: .trailing)
            let cutCount = str.count - strNew.count
            if cutCount > 0 {
                // 如果右侧有空白内容
                let replaceStr = strNew + Array(0..<cutCount).map{ _ in withText }.joined()
                listNew.append(replaceStr)
            } else {
                listNew.append(str)
            }
        }
        
        let text2 = listNew.joined(separator: "\n")
        return text2
    }
}

```
swift测试结果如下：
```
let text = " ab c "
// 只去掉空格
let trimLeftText = text.trimWhitespace(type: .leading)
let trimRightText = text.trimWhitespace(type: .trailing)
let trimBothText = text.trimWhitespace(type: .leadingAndTrailing)
let trimAllText = text.trimWhitespace(type: .all)
print("text: \(text)") // " ab c "
print("trimLeftText: \(trimLeftText)") // "ab c "
print("trimRightText: \(trimRightText)") // " ab c"
print("trimBothText: \(trimBothText)") // "ab c"
print("trimAllText: \(trimAllText)") // "abc"
```

----

项目中使有部分UI涉及到根据字符串的`sizeThatFits`计算出宽高\
在正常的计算没有任何问题，但是如果存在一个字符串存在换行符`\n`，同时在换行符的前面有几个空格，计算的宽度是去掉了空格的宽度，导致计算出的高度比真实的高度小，这种情况下需要将换行符前结尾的空白使用其他字符串代替进行计算，如下：
```
let label = UILabel()
let text = "古今中午   1233211   \n上下左右 1234567   \n其乐融融 1231231   "
label.attributedText = NSAttributedString(string: text, attributes: [NSAttributedString.Key.font : UIFont.systemFont(ofSize: 14)])
let maxWidth: CGFloat = 200
let size = label.sizeThatFits(CGSize(width: maxWidth, height: CGFloat.greatestFiniteMagnitude))
label.frame = CGRect(origin: CGPoint(x: 10, y: 100), size: size)

let label2 = UILabel()
// 将换行符前面的结尾部分的空格使用1来代替方便计算宽高
let text2 = text.replaceTrailingSpace()
label2.attributedText = NSAttributedString(string: text2, attributes: [NSAttributedString.Key.font : UIFont.systemFont(ofSize: 14)])
let size2 = label2.sizeThatFits(CGSize(width: maxWidth, height: CGFloat.greatestFiniteMagnitude))
label2.frame = CGRect(origin: CGPoint(x: 10, y: 200), size: size2)
// 计算完高度后，再填充原始的数据
label2.attributedText = NSAttributedString(string: text, attributes: [NSAttributedString.Key.font : UIFont.systemFont(ofSize: 14)])
```

产生这个问题的原因是使用了`M80AttributedLabel`这个框架，设置了行间距，在计算高度时没有行间距，但是显示时有行间距导致
```
let lab = M80AttributedLabel()
lab.backgroundColor = UIColor.clear
lab.font = UIFont.systemFont(ofSize: 16, weight: .regular)
lab.lineSpacing = 1
lab.numberOfLines = 0
// 由于这里设置的attributed没有设置linespace，会导致计算和展示的高度不一致
lab.appendAttributedText(xxxx)
lab.sizeThatFits(CGSize(width: 100, height: CGFloat.greatestFiniteMagnitude))
```

----
使用CTFramesetterSuggestFrameSizeWithConstraints计算宽高
```
@implementation UILabel (size)
- (CGSize)calculateSize:(CGSize)size
{
   
    NSAttributedString *drawString = self.attributedText;
    if (drawString == nil)
    {
        return CGSizeZero;
    }
    CFAttributedStringRef attributedStringRef = (__bridge CFAttributedStringRef)drawString;
    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString(attributedStringRef);
    CFRange range = CFRangeMake(0, 0);
    if (self.numberOfLines > 0 && framesetter)
    {
        CGMutablePathRef path = CGPathCreateMutable();
        CGPathAddRect(path, NULL, CGRectMake(0, 0, size.width, size.height));
        CTFrameRef frame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, 0), path, NULL);
        CFArrayRef lines = CTFrameGetLines(frame);
        
        if (nil != lines && CFArrayGetCount(lines) > 0)
        {
            NSInteger lastVisibleLineIndex = MIN(self.numberOfLines, CFArrayGetCount(lines)) - 1;
            CTLineRef lastVisibleLine = CFArrayGetValueAtIndex(lines, lastVisibleLineIndex);
            
            CFRange rangeToLayout = CTLineGetStringRange(lastVisibleLine);
            range = CFRangeMake(0, rangeToLayout.location + rangeToLayout.length);
        }
        CFRelease(frame);
        CFRelease(path);
    }
    
    CFRange fitCFRange = CFRangeMake(0, 0);
    CGSize newSize = CTFramesetterSuggestFrameSizeWithConstraints(framesetter, range, NULL, size, &fitCFRange);
    if (framesetter)
    {
        CFRelease(framesetter);
    }
    CGFloat width = ceilf(newSize.width) + 1;
    CGFloat height = MIN(ceilf(newSize.height) + 1, size.height);
    
    //高度超过单行高度的1.5倍, 视为换行了, 最优宽度达到最大宽度的0.9, 则使用最大宽度
    if (height > self.font.lineHeight * 1.5 && width > size.width * 0.9) {
        width = size.width;
    }
    return CGSizeMake(width, height);
}
@end
```
![image](/assets/img/other/other_string_size.png)