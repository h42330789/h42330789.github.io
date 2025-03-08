---
title: swift使用JSONSerialization.jsonObject丢失小数位精度问题
author: 独孤流
date: 2025-02-25 10:14:00 +0800
categories: [other, 其他]
tags: [iOS]     # TAG names should always be lowercase
---

> ### 前言
> 平常从接口请求到json数据后，一般都是使用`JSONSerialization.jsonObject(with: data, options:[])`的方法解析数据，但是一些需要保留小数位的场景下，数字都会自动转换为`Int32`、`Int`、`Int64`、`Double`类型，导致再将`Double`转换为字符串展示或参与数据计算时，存在精度丢失，或小数位为0时小数位丢失的问题，本来这种要保留格式化的小数位最好是接口返回成字符串，但因为一些特殊原因，后端接口只做了格式化，保留了指定的小数位，但是返回的任然是数字格式


### 解决方案一：
由于接口返回的数据不管什么格式，终究解析出来是字符串，在`JSONSerialization.jsonObject(with: data, options:[])`使用产生小数位丢失或精度问题，所以在解析成json之前，先把所有数字都包上引号`"`, 处理完后再使用`JSONSerialization`解析成JSON对象，但是这种方式有个问题，处理不严谨，会把所有数字都加上引号，导致读取时没处理好会读取失败，需要配合一个`NSDictionary`的分类来读取内容做兼容处理，具体代码如下

```
extension String {
    var convertNumbersToString: String {
        let json = self
        var result = ""
        var index = json.startIndex
        var insideString = false // 是否在字符串内部
        
        while index < json.endIndex {
            let char = json[index]
            
            // 处理字符串，跳过其中的内容
            if char == "\"" {
                insideString.toggle() // 进入或退出字符串
                result.append(char)
                index = json.index(after: index)
                continue
            }
            
            // 如果当前不在字符串内，查找数字
            if !insideString {
                // 检查是否是数字的开始（包括负号和小数点）
                if char == "-" || char.isNumber {
                    result.append("\"") // 加前引号
                    result.append(char)
                    index = json.index(after: index)
                    
                    // 继续追加数字字符
                    while index < json.endIndex, json[index].isNumber || json[index] == "." {
                        result.append(json[index])
                        index = json.index(after: index)
                    }
                    
                    result.append("\"") // 加后引号
                    continue
                }
            }
            
            // 其他情况，直接追加字符
            result.append(char)
            index = json.index(after: index)
        }
        
        return result
    }
}
```

`String`、`Int`、`Int`、`Int64`、`Bool`、`Dictionary`使用扩展获取获取内容


```
// MARK: 各类工具扩展
extension String {
    enum IntRoundType {
        case down
        case up
        case round
    }
    func toInt(type: IntRoundType = .down)-> Int{
        switch type {
        case .down:
            return Int(floor(doubleValue)) // 向下取整
        case .up:
            return Int(ceil(doubleValue)) // 向上取整
        case .round:
            return Int(round(doubleValue)) // 四舍五入取整
        }
    }
    
    func toInt32(type: IntRoundType = .down)-> Int32{
        switch type {
        case .down:
            return Int32(floor(doubleValue)) // 向下取整
        case .up:
            return Int32(ceil(doubleValue)) // 向上取整
        case .round:
            return Int32(round(doubleValue)) // 四舍五入取整
        }
    }
    
    func toInt64(type: IntRoundType = .down)-> Int64{
        switch type {
        case .down:
            return Int64(floor(doubleValue)) // 向下取整
        case .up:
            return Int64(ceil(doubleValue)) // 向上取整
        case .round:
            return Int64(round(doubleValue)) // 四舍五入取整
        }
    }
    
    func toCGFloat()-> CGFloat{
        return CGFloat(self.toDouble())
    }
    
    func toDouble()-> Double{
        return self.doubleValue
    }
    var doubleValue: Double {
        return Double(self.numberText) ?? 0
    }
    var decimalNumber: NSDecimalNumber {
        let valStr = self.numberText
        if valStr.isBlank {
            return NSDecimalNumber.zero
        }
        return NSDecimalNumber(string: valStr)
    }
    var numberText: String {
        var txt = self.replace(",", with: "")
        txt = txt.replace("，", with: "")
        txt = txt.replace("%", with: "")
        txt = txt.replace("￥", with: "")
        txt = txt.replace("¥", with: "")
        txt = txt.replace("$", with: "")
        txt = txt.replace(" ", with: "")
        return txt
    }
    func replace(_ string: String, with withString: String) -> String {
        return replacingOccurrences(of: string, with: withString)
    }
    var isBlank: Bool {
        let trimmed = trimmingCharacters(in: .whitespacesAndNewlines)
        return trimmed.isEmpty == true
    }
    
}
extension Int64 {
    var strValue: String {
        return "\(self)"
    }
    var int32Val: Int32 {
        return Int32(self)
    }
}

extension Int32 {
    var strValue: String {
        return Int64(self).strValue
    }
}
extension Int {
    var strValue: String {
        return Int64(self).strValue
    }
    func strCountStr(max: Int = 99, moreStr: String = "+") -> String {
        if self < max {
            return self.strValue
        }
        return "\(max)\(moreStr)"
    }
    var int32Val: Int32 {
        return Int32(self)
    }
}

extension Bool {
    var strValue: String {
        return self == true ? "1" : "0"
    }
}
// MARK: - 从字典里获取值的扩展
extension Dictionary where Key == String, Value == Any {
    func getString(_ key: String) -> String? {
        guard let val = self[key] else {
            return nil
        }
        if let strVal = val as? String {
            return strVal
        }
        if let intVal = val as? Int64 {
            return intVal.strValue
        }
        if let intVal = val as? Int {
            return intVal.strValue
        }
        if let intVal = val as? Int32 {
            return intVal.strValue
        }
        if let dVal = val as? Double {
            let strV = "\(dVal)"
            return strV
        }
        return nil
    }
    func getInt64(_ key: String) -> Int64? {
        guard let val = self[key] else {
            return nil
        }
        if let intVal = val as? Int64 {
            return intVal
        }
        if let strVal = val as? String {
            return strVal.toInt64()
        }
        if let intVal = val as? Int {
            return Int64(intVal)
        }
        if let intVal = val as? Int32 {
            return Int64(intVal)
        }
        if let dVal = val as? Double {
            return Int64(dVal)
        }
        return nil
    }
    func getInt(_ key: String) -> Int? {
        guard let val = self[key] else {
            return nil
        }
        if let intVal = val as? Int {
            return intVal
        }
        if let strVal = val as? String {
            return strVal.toInt()
        }
        if let intVal = val as? Int64 {
            return Int(intVal)
        }
        if let intVal = val as? Int32 {
            return Int(intVal)
        }
        if let dVal = val as? Double {
            return Int(dVal)
        }
        return nil
    }
    func getInt32(_ key: String) -> Int32? {
        guard let val = self[key] else {
            return nil
        }
        if let intVal = val as? Int32 {
            return intVal
        }
        if let strVal = val as? String {
            return strVal.toInt32()
        }
        if let intVal = val as? Int64 {
            return Int32(intVal)
        }
        if let intVal = val as? Int {
            return Int32(intVal)
        }
        if let dVal = val as? Double {
            return Int32(dVal)
        }
        return nil
    }
    func getDouble(_ key: String) -> Double? {
        guard let val = self[key] else {
            return nil
        }
        if let intVal = val as? Double {
            return intVal
        }
        if let strVal = val as? String {
            return strVal.toDouble()
        }
        if let intVal = val as? Int64 {
            return Double(intVal)
        }
        if let intVal = val as? Int {
            return Double(intVal)
        }
        if let dVal = val as? Int32 {
            return Double(dVal)
        }
        return nil
    }
    func getBool(_ key: String) -> Bool? {
        guard let val = self[key] else {
            return nil
        }
        if let intVal = val as? Bool {
            return intVal
        }
        if let strVal = val as? String {
            let lowStr = strVal.lowercased()
            if  lowStr == "true" || lowStr == "1" {
                return true
            }
            if  lowStr == "false" || lowStr == "0" {
                return false
            }
            return false
        }
        if let intVal = val as? Int64 {
            return intVal == 1
        }
        if let intVal = val as? Int {
            return intVal == 1
        }
        if let dVal = val as? Int32 {
            return dVal == 1
        }
        return false
    }
    func getInt32List(_ key: String) -> [Int32]? {
        guard let val = self[key] else {
            return nil
        }
        guard let listVal = val as? [Any] else {
            return []
        }
        let list = listVal.compactMap{ ["key": $0].getInt32("key") }
        return list
    }
    func getIntList(_ key: String) -> [Int]? {
        guard let val = self[key] else {
            return nil
        }
        guard let listVal = val as? [Any] else {
            return []
        }
        let list = listVal.compactMap{ ["key": $0].getInt("key") }
        return list
    }
    func getInt64List(_ key: String) -> [Int64]? {
        guard let val = self[key] else {
            return nil
        }
        guard let listVal = val as? [Any] else {
            return []
        }
        let list = listVal.compactMap{ ["key": $0].getInt64("key") }
        return list
    }
    func getDoubleList(_ key: String) -> [Double]? {
        guard let val = self[key] else {
            return nil
        }
        guard let listVal = val as? [Any] else {
            return []
        }
        let list = listVal.compactMap{ ["key": $0].getDouble("key") }
        return list
    }
    func getBoolList(_ key: String) -> [Bool]? {
        guard let val = self[key] else {
            return nil
        }
        guard let listVal = val as? [Any] else {
            return []
        }
        let list = listVal.compactMap{ ["key": $0].getBool("key") }
        return list
    }
    func getStringList(_ key: String) -> [String]? {
        guard let val = self[key] else {
            return nil
        }
        guard let listVal = val as? [Any] else {
            return []
        }
        let list = listVal.compactMap{ ["key": $0].getString("key") }
        return list
    }
}

```

调用例子
```
let str = "{\"name\":\"jim\",\"age\":10, \"height\": 1.650, \"payList\": [{\"fee:\": -40.56},{\"fee:\": -44.00},{\"fee:\": -44.40},{\"fee:\": -42.00},{\"fee:\": 0.00}]}"
guard let data = str.convertNumbersToString.data(using: String.Encoding.utf8) else {
    return nil
}

guard let dict = try? JSONSerialization.jsonObject(with: data, options: []) as? [String: Any] else {
    return nil
}
print(dict)
let name = dict?.getString("name")
let age = dict?.getInt32("age")
let height = dict?.getString("height")
```
----
### ~~方案二、不使用系统的`JSONSerialization.jsonObject`,自己写解析JSON字符串的类,遇到数字时，如果有小数点，都转换成字符串，这样会更灵活~~

##### ~~2.1、使用正则将原始字符串中的数字全部补上双引号，由于场景覆盖不全，且一些没有小数位的也处理了，容易处理出错以及复杂，已废弃~~
```
// 正则表达式匹配 JSON 中的浮点数（带小数位的数字）
static func convertFloatsToStrings(in jsonString: String?) -> String? {
    guard let jsonString = jsonString, !jsonString.isEmpty else {
        return jsonString
    }
    
    // 正则表达式匹配 JSON 中的浮点数
    let pattern = #"-?\d+\.\d+"#
    
    do {
        let regex = try NSRegularExpression(pattern: pattern, options: [])
        
        // 替换匹配的浮点数为带双引号的字符串
        var modifiedString = jsonString
        let matches = regex.matches(in: jsonString, options: [], range: NSRange(location: 0, length: jsonString.utf16.count))
        
        // 从后往前替换，避免影响后续匹配的范围
        for match in matches.reversed() {
            if let range = Range(match.range, in: jsonString) {
                let matchedString = String(jsonString[range])
                let replacement = "\"\(matchedString)\""
                modifiedString.replaceSubrange(range, with: replacement)
            }
        }
        
        return modifiedString
    } catch {
        print("正则表达式错误: \(error)")
        return jsonString
    }

}
// 正则表达式匹配 JSON 中的浮点数（带小数位的数字）
func convertFloatsToStrings2(in jsonString: String?) -> String? {
    guard let jsonString = jsonString, jsonString.count > 0 else {
        return jsonString
    }
    let pattern = #"(\s*:\s*)(\d+\.\d+)(\s*[,}\]])"#
    let regex = try! NSRegularExpression(pattern: pattern, options: [])
    
    // 替换匹配的浮点数为带双引号的字符串
    let modifiedString = regex.stringByReplacingMatches(
        in: jsonString,
        options: [],
        range: NSRange(location: 0, length: jsonString.utf16.count),
        withTemplate: "$1\"$2\"$3"
    )
    
    return modifiedString
}

```

##### ~~2.2 将原始字符串自定义json解析（已废弃）~~
这个自定义的写的不够完善，遇到一些emoji或Unicode码会导致抛出异常

```
enum JSONError: Error {
    case invalidJSON
    case unexpectedCharacter(Character, Int)
    case unexpectedEndOfInput
}

class JSONParser {
    private let json: String
    private var index: String.Index
    
    init(json: String) {
        self.json = json
        self.index = json.startIndex
    }
    
    func parse() throws -> Any {
        let value = try parseValue()
        skipWhitespace()
        if index != json.endIndex {
            throw JSONError.invalidJSON
        }
        return value
    }
    
    private func parseValue() throws -> Any {
        skipWhitespace()
        guard index < json.endIndex else {
            throw JSONError.unexpectedEndOfInput
        }
        
        let char = json[index]
        
        switch char {
        case "{": return try parseObject()
        case "[": return try parseArray()
        case "\"": return try parseString()
        case "t", "f": return try parseBoolean()
        case "n": return try parseNull()
        case "-", "0"..."9": return try parseNumber()
        default:
            throw JSONError.unexpectedCharacter(char, json.distance(from: json.startIndex, to: index))
        }
    }
    
    private func parseObject() throws -> [String: Any] {
        var object: [String: Any] = [:]
        advance() // 跳过 `{`
        skipWhitespace()
        
        while index < json.endIndex, json[index] != "}" {
            let key = try parseString()
            skipWhitespace()
            
            guard index < json.endIndex, json[index] == ":" else {
                throw JSONError.invalidJSON
            }
            advance() // 跳过 `:`
            skipWhitespace()
            
            let value = try parseValue()
            object[key] = value
            
            skipWhitespace()
            if index < json.endIndex, json[index] == "," {
                advance() // 跳过 `,`
                skipWhitespace()
            } else {
                break
            }
        }
        
        guard index < json.endIndex, json[index] == "}" else {
            throw JSONError.invalidJSON
        }
        advance() // 跳过 `}`
        
        return object
    }
    
    private func parseArray() throws -> [Any] {
        var array: [Any] = []
        advance() // 跳过 `[`
        skipWhitespace()
        
        while index < json.endIndex, json[index] != "]" {
            let value = try parseValue()
            array.append(value)
            
            skipWhitespace()
            if index < json.endIndex, json[index] == "," {
                advance() // 跳过 `,`
                skipWhitespace()
            } else {
                break
            }
        }
        
        guard index < json.endIndex, json[index] == "]" else {
            throw JSONError.invalidJSON
        }
        advance() // 跳过 `]`
        
        return array
    }
    
    private func parseString() throws -> String {
        advance() // 跳过 `"`
        var result = ""
        
        while index < json.endIndex {
            let char = json[index]
            if char == "\"" {
                advance() // 跳过 `"`
                return result
            } else if char == "\\" {
                advance() // 跳过 `\`
                if index < json.endIndex {
                    let escapedChar = json[index]
                    switch escapedChar {
                    case "\"": result.append("\"")
                    case "\\": result.append("\\")
                    case "/": result.append("/")
                    case "b": result.append("\u{08}")
                    case "f": result.append("\u{0C}")
                    case "n": result.append("\n")
                    case "r": result.append("\r")
                    case "t": result.append("\t")
                    default:
                        throw JSONError.unexpectedCharacter(escapedChar, json.distance(from: json.startIndex, to: index))
                    }
                }
            } else {
                result.append(char)
            }
            advance()
        }
        
        throw JSONError.unexpectedEndOfInput
    }
    
    private func parseBoolean() throws -> Bool {
        if json[index...].hasPrefix("true") {
            advance(by: 4) // 跳过 `true`
            return true
        } else if json[index...].hasPrefix("false") {
            advance(by: 5) // 跳过 `false`
            return false
        } else {
            throw JSONError.invalidJSON
        }
    }
    
    private func parseNull() throws -> Any {
        if json[index...].hasPrefix("null") {
            advance(by: 4) // 跳过 `null`
            return NSNull()
        } else {
            throw JSONError.invalidJSON
        }
    }
    
    private func parseNumber() throws -> Any {
        var numString = ""
        var hasDecimal = false
        
        while index < json.endIndex {
            let char = json[index]
            if char.isNumber || char == "-" {
                numString.append(char)
            } else if char == "." {
                if hasDecimal {
                    throw JSONError.invalidJSON // 多个 `.` 不合法
                }
                hasDecimal = true
                numString.append(char)
            } else {
                break
            }
            advance()
        }
        if numString.contains(".") {
            // 有小数位返回字符串
            return numString // 保持字符串格式，防止丢失精度
        } else {
            let val = numString.toInt64()
            if val <= Int64(Int32.max) {
                return numString.toInt32()
            } else if val <= Int64(Int.max) {
                return numString.toInt()
            }
            return val // 保持字符串格式，防止丢失精度
        }
        
    }
    
    // 跳过空格
    private func skipWhitespace() {
        while index < json.endIndex, json[index].isWhitespace {
            advance()
        }
    }
    
    // 前进一个字符
    private func advance() {
        index = json.index(after: index)
    }
    
    // 前进 `n` 个字符
    private func advance(by count: Int) {
        index = json.index(index, offsetBy: count, limitedBy: json.endIndex) ?? json.endIndex
    }
}
```

##### 2.3 使用开源的项目修改（推荐）
- [https://github.com/swiftdo/json](https://github.com/swiftdo/json)
- [Swift 码了个 JSON 解析器(一)](https://oldbird.run/swift/fp/t3-json1.html)
- [Swift 码了个 JSON 解析器(二)](https://oldbird.run/swift/fp/t3-json2.html)
- [Swift 码了个 JSON 解析器(三)](https://oldbird.run/swift/fp/t3-json3.html)

```
import Foundation


extension StringProtocol {
    subscript(offset: Int) -> Character {
        if offset > count {
            fatalError("offset 越界:\(offset)")
        }
        let c = self[index(startIndex, offsetBy: offset)]
        return c
    }
}


class JSONParserNew {
    // Swift 码了个 JSON 解析器(一) https://oldbird.run/swift/fp/t3-json1.html
    // Swift 码了个 JSON 解析器(二) https://oldbird.run/swift/fp/t3-json2.html
    // Swift 码了个 JSON 解析器(三) https://oldbird.run/swift/fp/t3-json3.html

    // 解析原理：
    // * 解析对象 {}
    //   对象结构是 `{"Key": [值]}` 的格式，所以先解析到Key字符串，将Key 解析出来，然后再解析值，因为值有可能 [`字符串`、`值类型`、`布尔类型`、`对象`、`数组`、`null`] 所以需要根据前缀得到类型，并调用相应的解析方法，循环解析到 `}` 对象的结尾
    // * 解析数组 []
    //   对象的结构是 `[[值]、[值]]`，因为值有可能是 [`字符串`、`值类型`、`布尔类型`、`对象`、`数组`、`null`] 所以需要根据前缀得到类型，并调用相应的解析方法，循环解析到 `}` 对象的结尾
    // * 解析字符串
    //   循环解析，需要判断是否遇到转义符`\`如果遇到，当前字符的下一个字符将是作为普通字符存入结果，如果遇到非转义的 " 字符则退出字符串读取方法，并返回结果
    // * 解析值类型
    //   循环拉取[0-9]包括.符号，然后调用转换成double类型方法
    // * 解析布尔类型
    //   转判断是 true 还是 false
    // * 解析null
    //   转判断是否为 null

    struct ParserError: Error, CustomStringConvertible {
        let message: String
        
        init(msg: String) {
            self.message = msg
        }
        
        var description: String {
            return "ParserError: \(message)"
        }
    }
    var originString: String = ""
    var double2String: Bool = true
    var openScience: Bool = true
    init(double2String: Bool = true, openScience: Bool = true) {
        self.double2String = double2String
        self.openScience = openScience
    }
    /// 解析 JSON 字符串为 JSON
    func parseJson(str: String?) throws -> Any {
        guard let str = str else {
            throw ParserError(msg: "json字符串为空")
        }
        var index = 0
        // 读取非空白字符
        index = readToNonBlankIndex(str: str, index: index)

        let ch = str[index]
        index += 1

        if ch == "[" {
            // 解析数组
            return try parseArray(str: str, index: index).0
        }

        // 解析对象
        return try parseObject(str: str, index: index).0
    }
    
    /// 解析JSON字符串为对象结构
    func parseObject(str: String, index: Int) throws -> (Any, Int) {
        var ind = index
        var obj: [String: Any] = [:]

        repeat {
            ind = readToNonBlankIndex(str: str, index: ind)
            if str[ind] != "\"" {
                throw ParserError(msg: "不能识别的字符“\(str[ind]) ind:\(ind) str:\(str)”")
            }
            ind += 1

            // 读取字符串
            let (name, ids) = try readString(str: str, index: ind)
            ind = ids

            if obj.keys.contains(name) {
                throw ParserError(msg: "已经存在key: \(name) ind:\(ind) str:\(str)")
            }
            ind = readToNonBlankIndex(str: str, index: ind)

            if str[ind] != ":" {
                throw ParserError(msg: "不能识别字符:\(str[ind]) ind:\(ind) str:\(str)")
            }

            ind += 1
            ind = readToNonBlankIndex(str: str, index: ind)

            /// 读取下一个 element
            let next = try readElement(str: str, index: ind)
            ind = next.1
            obj[name] = next.0

            /// 读取到非空白字符
            ind = readToNonBlankIndex(str: str, index: ind)

            let ch = str[ind]
            ind += 1
            if ch == "}" { break }
            if ch != "," {
                throw ParserError(msg: "不能识别的字符 ind:\(ind) str:\(str)")
            }
        } while true

        return (obj, ind)
    }

    /// 解析JSON字符串为数组结构
    func parseArray(str: String, index: Int) throws -> (Any, Int) {
        var arr: [Any] = []
        var ind = index
        repeat {
            ind = readToNonBlankIndex(str: str, index: ind)
            /// 读取下一个element
            let ele = try readElement(str: str, index: ind)
            ind = ele.1
            arr.append(ele.0)
            /// 读取非空白字符
            ind = readToNonBlankIndex(str: str, index: ind)

            let ch = str[ind]
            ind += 1
            if ch == "]" { break }
            if ch != "," {
                throw ParserError(msg: "不能识别的字符 ind:\(ind) str:\(str)")
            }
        } while true

        return (arr, ind)
    }

    /// 读取下一个
    func readElement(str: String, index: Int) throws -> (Any, Int) {
        var ind = index
        let c = str[ind]
        ind += 1
        switch c {
        case "[":
            return try parseArray(str: str, index: ind)
        case "{":
            return try parseObject(str: str, index: ind)
        case "\"":
            let (str, ids) = try readString(str: str, index: ind)
            return (str, ids)
        case "t":
            return try readJsonTrue(str: str, index: ind)
        case "f":
            return try readJsonFalse(str: str, index: ind)
        case "n":
            return try readJsonNull(str: str, index: ind)
        case _ where isNumber(c: c):
            return try readJsonNumber(str: str, index: ind)
        default:
            throw ParserError(msg: "未知 element: \(c) \(ind) \(str)")
        }
    }

    func readJsonNumber(str: String, index: Int) throws -> (Any, Int) {
        var ind = index - 1
        var value: [Character] = []
        
        while ind < str.count && isNumber(c: str[ind]) {
            value.append(str[ind])
            ind += 1
        }
        var str = String(value)
        // 科学计数法
        // 123.4e5 = 123.4 -> 小数点往后移5位
        // 123.4e-5 = 123.4*0.00001 -> 小数点往前移动5位
        if value.count >= 3, value[value.count-2] == "e" {
//            let number1 = NSDecimalNumber(string: str)
//            print(number1)
            // 小数点往后移动
        } else if value.count >= 4, value[value.count-3] == "e", value[value.count-2] == "-" {
            // 小数点往前移动
            if openScience {
                let number1 = NSDecimalNumber(string: str)
                str = "\(number1)"
            }
            
        }
        if str.contains(".") {
            if double2String {
                if str.contains("e"), let dval = Double(str) {
                    str = "\(dval)"
                }
                return (str, ind)
            } else {
                if let v = Double(str) {
                    return (v, ind)
                }
            }
            
        } else {
            if let val = Int64(str) {
                if val <= Int64(Int32.max) {
                    // 如果是Int32以内，使用Int32
                    return (Int32(val), ind)
                } else if val <= Int64(Int.max) {
                    // 如果是Int范围使用Int
                    return (Int(val), ind)
                }
                return (val, ind)
            }
        }
        throw ParserError(msg: "不能识别的数字类型\(ind) ind:\(ind) \(str)")
    }


    func readJsonNull(str: String, index: Int) throws -> (Any, Int) {
        return try readJsonCharacters(str: str,
                                      index: index,
                                      characters: ["u", "l", "l"],
                                      error: ParserError(msg: "读取null值出错 ind:\(index) str:\(str)"),
                                      json: NSNull())
    }

    func readJsonFalse(str: String, index: Int) throws -> (Any, Int) {
        return try readJsonCharacters(str: str,
                                      index: index,
                                      characters: ["a", "l", "s", "e"],
                                      error: ParserError(msg: "读取false值出错 ind:\(index) str:\(str)"),
                                      json: false)
    }

    func readJsonTrue(str: String, index: Int) throws -> (Any, Int) {
        return try readJsonCharacters(str: str,
                                      index: index,
                                      characters: ["r", "u", "e"],
                                      error: ParserError(msg: "读取true值出错 ind:\(index) str:\(str)"),
                                      json: true)
    }

    /// 读取字符串
    func readString(str: String, index: Int) throws -> (String, Int) {
        var ind = index
        var value: [Character] = []

        while ind < str.count {
            
            var c = str[ind]
            ind += 1
            
            if c == "\\" { // 判断是否是转义字符
                value.append("\\")
                if ind >= str.count {
                    throw ParserError(msg: "未知结尾 ind:\(ind) str:\(str)")
                }

                c = str[ind]
                ind += 1
                value.append(c)

                if c == "u" {
                    try (0 ..< 4).forEach { _ in
                        c = str[ind]
                        ind += 1

                        if isHex(c: c) {
                            value.append(c)
                        } else {
                            throw ParserError(msg: "不是有效的unicode 字符 ind:\(ind) str:\(str)")
                        }
                    }
                }
            } else if c == "\"" {
                break
            } else if c == "\r" || c == "\n" {
                throw ParserError(msg: "传入的JSON字符串内容不允许有换行 ind:\(ind) str:\(str)")
            } else {
                value.append(c)
            }
            
        }

        return (String(value), ind)
    }


    /// 读取到非空白字符, index 指向的是非空白字符的位置
    func readToNonBlankIndex(str: String, index: Int) -> Int {
        var ind = index
        while ind < str.count && str[ind] == " " {
            ind += 1
        }
        return ind
    }

    /// tools
    func readJsonCharacters(str: String, index: Int, characters: [Character], error: ParserError, json: Any) throws -> (Any, Int) {
        var ind = index
        var result = true
        
        for i in 0 ..< characters.count {
            if str.count <= ind || str[ind] != characters[i] {
                result = false
                break
            }
            ind += 1
        }
        if result {
            return (json, ind)
        }
        throw error
    }

    /// 判断是否是数字字符
    func isNumber(c: Character) -> Bool {
        
        let chars:[Character: Bool] = ["-": true, "+": true, "e": true, "E": true, ".": true]
        
        if let b = chars[c], b {
            return true
        }
        
        if(c >= "0" && c <= "9") {
            return true
        }
        
        return false;
    }

    /// 判断是否为 16 进制字符
    func isHex(c: Character) -> Bool {
        return c >= "0" && c <= "9" || c >= "a" && c <= "f" || c >= "A" && c <= "F"
    }


}

```



使用方式

```
let str = "{\"name\":\"jim\",\"age\":10, \"height\": 1.650, \"payList\": [{\"fee:\": -40.56},{\"fee:\": -44.00},{\"fee:\": -44.40},{\"fee:\": -42.00},{\"fee:\": 0.00}]}"
let jsonString = "{\"emoji\": \"😀👨‍👩‍👧‍👦🌍\",\"unicode\": \"\\uD83D\\uDE00\",\"name\":\"Byby&$%+~\\u003C\\u003E\",\"price\": 189, \"a1\": 1.23e4, \"a2\": 12e-4}"

if let data = str.data(using: String.Encoding.utf8),
    let dict1 = try? JSONSerialization.jsonObject(with: data, options: []) as? [String: Any] {
    let name1 = dict1.getString("name")
    let age1 = dict1.getInt32("age")
    let height1 = dict1.getString("height")
    print(dict1)
    print("name1: \(name1 ?? "") age1: \(age1 ?? 0) height1: \(height1 ?? "")\n")
}
if let data2 = str.convertNumbersToString.data(using: String.Encoding.utf8)
    , let dict2 = try? JSONSerialization.jsonObject(with: data2, options: []) as? [String: Any] {
    
    let name2 = dict2.getString("name")
    let age2 = dict2.getInt32("age")
    let height2 = dict2.getString("height")
    print(dict2)
    print("name2: \(name2 ?? "") age2: \(age2 ?? 0) height2: \(height2 ?? "")\n")
}

do {
    let parser = JSONParser(json: str)
    let dict3 = try parser.parse() as? [String: Any]
    let name3 = dict3?.getString("name")
    let age3 = dict3?.getInt32("age")
    let height3 = dict3?.getString("height")
    print(dict3 ?? [:])
    print("name3: \(name3 ?? "") age3: \(age3 ?? 0) height3: \(height3 ?? "")\n")
} catch {
    print(error)
}

do {
    let parser = JSONParserNew()
    let dict4 = try parser.parseJson(str: str) as? [String: Any]
    let name4 = dict4?.getString("name")
    let age4 = dict4?.getInt32("age")
    let height4 = dict4?.getString("height")
    print(dict4 ?? [:])
    print("name4: \(name4 ?? "") age4: \(age4 ?? 0) height4: \(height4 ?? "")\n")
} catch {
    print(error)
}


```

![image](/assets/img/other/jsonDot.png)