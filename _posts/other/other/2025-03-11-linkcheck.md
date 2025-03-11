---
title: iOS检测文本里的链接、数字等的内容
author: 独孤流
date: 2025-03-11 10:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

> 常见的iOS应用里，一行文字都可以标记出电话号码、链接等内容，方便用户在一段文本中点击拨打电话或点击链接

系统级实现方式,使用系统的\
`NSDataDetector` + `NSTextCheckingTypeLink` +`NSTextCheckingTypePhoneNumber`

`- (void)enumerateMatchesInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range usingBlock:(void (NS_NOESCAPE ^)(NSTextCheckingResult * _Nullable result, NSMatchingFlags flags, BOOL *stop))block`

具体获取和实现代码
```
NSString *plainText = @let text = "联系👏🏻 订单1234567 👋 我👨‍👩‍👧‍👦：123-456-7890 或访问 https://example.com 获取更多 google.com信息。";
// 获取检测器
NSDataDetector *detector = [NSDataDetector dataDetectorWithTypes:NSTextCheckingTypeLink | NSTextCheckingTypePhoneNumber
                                                   error:nil];
// 使用检测器检测文本内容
[detector enumerateMatchesInString:plainText
                           options:0
                             range:NSMakeRange(0, [plainText length])
                        usingBlock:^(NSTextCheckingResult * _Nullable result, NSMatchingFlags flags, BOOL * _Nonnull stop) {
    NSRange range = result.range;
    NSString *subString = [plainText substringWithRange:range];
    let resultType = result.resultType;
    NSLog(@"range: %@ type:%llu text: %@ url: %@", NSStringFromRange(range), result.resultType, subString, result.URL);
}];
```
swift 版本的写法
```
let text = "联系👏🏻 订单123456 👋 我👨‍👩‍👧‍👦：123-456-7890 或1234567 和 1234568 访问 https://example.com 获取更多 google.com信息。"
print(text)
print("NSString(text).length: \(NSString(string: text).length)")
print("text.count: \(text.count)")
print("text.utf8.count: \(text.utf8.count)")
print("text.utf16.count: \(text.utf16.count)")
// String的utf16.count 与 NSString的length是一样的
do {
    // 配置查找规则
    let detector = try? NSDataDetector(types: NSTextCheckingResult.CheckingType.link.rawValue | NSTextCheckingResult.CheckingType.phoneNumber.rawValue)
    // 在指定字符串里查找匹配
    let results = detector?.matches(in: text, range: NSRange(location: 0, length: text.utf16.count)) ?? []
    // 处理查找结果
    for result in results {
        // 将range反向查找字符串
        var subString: String? = nil
        if let textRange = Range(result.range, in: text) {
            subString = String(text[textRange])
        }
        print("range: \(result.range) type:\(result.resultType) text: \(subString ?? "") url:\(result.url?.absoluteString ?? "")")
    }
}
/** 
联系👏🏻 订单12345 👋 我👨‍👩‍👧‍👦：123-456-7890 或1234567 和 1234568 访问 https://example.com 获取更多 google.com信息。
NSString(text).length: 104
text.count: 90
text.utf8.count: 158
text.utf16.count: 104
range: {31, 12} type:NSTextCheckingType(rawValue: 2048) text: 123-456-7890 url:
range: {45, 7} type:NSTextCheckingType(rawValue: 2048) text: 1234567 url:
range: {55, 7} type:NSTextCheckingType(rawValue: 2048) text: 1234568 url:
range: {66, 19} type:NSTextCheckingType(rawValue: 32) text: https://example.com url:https://example.com
range: {91, 10} type:NSTextCheckingType(rawValue: 32) text: google.com url:http://google.com
*/
```


----
支持的其他类型如下：
```
ypedef NS_OPTIONS(uint64_t, NSTextCheckingType) {    // a single type
    NSTextCheckingTypeOrthography           = 1ULL << 0,            // language identification
    NSTextCheckingTypeSpelling              = 1ULL << 1,            // spell checking
    NSTextCheckingTypeGrammar               = 1ULL << 2,            // grammar checking
    NSTextCheckingTypeDate                  = 1ULL << 3,            // date/time detection
    NSTextCheckingTypeAddress               = 1ULL << 4,            // address detection
    NSTextCheckingTypeLink                  = 1ULL << 5,            // link detection
    NSTextCheckingTypeQuote                 = 1ULL << 6,            // smart quotes
    NSTextCheckingTypeDash                  = 1ULL << 7,            // smart dashes
    NSTextCheckingTypeReplacement           = 1ULL << 8,            // fixed replacements, such as copyright symbol for (c)
    NSTextCheckingTypeCorrection            = 1ULL << 9,            // autocorrection
    NSTextCheckingTypeRegularExpression API_AVAILABLE(macos(10.7), ios(4.0), watchos(2.0), tvos(9.0))  = 1ULL << 10,           // regular expression matches
    NSTextCheckingTypePhoneNumber API_AVAILABLE(macos(10.7), ios(4.0), watchos(2.0), tvos(9.0))        = 1ULL << 11,           // phone number detection
    NSTextCheckingTypeTransitInformation API_AVAILABLE(macos(10.7), ios(4.0), watchos(2.0), tvos(9.0)) = 1ULL << 12            // transit (e.g. flight) info detection
};

extension NSTextCheckingResult {

    public struct CheckingType : OptionSet, @unchecked Sendable {

        public init(rawValue: UInt64)

        public static var orthography: NSTextCheckingResult.CheckingType { get }

        public static var spelling: NSTextCheckingResult.CheckingType { get }

        public static var grammar: NSTextCheckingResult.CheckingType { get }

        public static var date: NSTextCheckingResult.CheckingType { get }

        public static var address: NSTextCheckingResult.CheckingType { get }

        public static var link: NSTextCheckingResult.CheckingType { get }

        public static var quote: NSTextCheckingResult.CheckingType { get }

        public static var dash: NSTextCheckingResult.CheckingType { get }

        public static var replacement: NSTextCheckingResult.CheckingType { get }

        public static var correction: NSTextCheckingResult.CheckingType { get }

        @available(iOS 4.0, *)
        public static var regularExpression: NSTextCheckingResult.CheckingType { get }

        @available(iOS 4.0, *)
        public static var phoneNumber: NSTextCheckingResult.CheckingType { get }

        @available(iOS 4.0, *)
        public static var transitInformation: NSTextCheckingResult.CheckingType { get }
    }
}
```