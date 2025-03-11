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

##### ~~2.3 使用开源的项目修改（长json性能很差）~~
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

#### 2.4 使用开源的swift-corelibs-foundation框架（推荐）
- [swift-corelibs-foundation](https://github.com/swiftlang/swift-corelibs-foundation/tree/main)
- [JSONSerialization.swift](https://github.com/swiftlang/swift-corelibs-foundation/blob/main/Sources/Foundation/JSONSerialization.swift)
- [JSONSerialization+Parser.swift](https://github.com/swiftlang/swift-corelibs-foundation/blob/main/Sources/Foundation/JSONSerialization%2BParser.swift)

改写的完整代码如下：
```
//===----------------------------------------------------------------------===//
//
// This source file is part of the Swift.org open source project
//
// Copyright (c) 2014 - 2021 Apple Inc. and the Swift project authors
// Licensed under Apache License v2.0 with Runtime Library Exception
//
// See https://swift.org/LICENSE.txt for license information
// See https://swift.org/CONTRIBUTORS.txt for the list of Swift project authors
//
//===----------------------------------------------------------------------===//

import Foundation

extension JSONSerializationX {
    public struct ReadingOptions : OptionSet, Sendable {
        public let rawValue: UInt
        public init(rawValue: UInt) { self.rawValue = rawValue }
        
        public static let mutableContainers = ReadingOptions(rawValue: 1 << 0)
        public static let mutableLeaves = ReadingOptions(rawValue: 1 << 1)
        
        public static let fragmentsAllowed = ReadingOptions(rawValue: 1 << 2)
        @available(swift, deprecated: 100000, renamed: "JSONSerializationX.ReadingOptions.fragmentsAllowed")
        public static let allowFragments = ReadingOptions(rawValue: 1 << 2)
        
        public static let doubleToString = ReadingOptions(rawValue: 1 << 3)
    }

    public struct WritingOptions : OptionSet, Sendable {
        public let rawValue: UInt
        public init(rawValue: UInt) { self.rawValue = rawValue }
        
        public static let prettyPrinted = WritingOptions(rawValue: 1 << 0)
        public static let sortedKeys = WritingOptions(rawValue: 1 << 1)
        public static let fragmentsAllowed = WritingOptions(rawValue: 1 << 2)
        public static let withoutEscapingSlashes = WritingOptions(rawValue: 1 << 3)
    }
}

extension JSONSerializationX {
    // Structures with container nesting deeper than this limit are not valid if passed in in-memory for validation, nor if they are read during deserialization.
    // This matches Darwin Foundation's validation behavior.
    fileprivate static let maximumRecursionDepth = 512
}


/* A class for converting JSON to Foundation/Swift objects and converting Foundation/Swift objects to JSON.
   
   An object that may be converted to JSON must have the following properties:
    - Top level object is a `Swift.Array` or `Swift.Dictionary`
    - All objects are `Swift.String`, `Foundation.NSNumber`, `Swift.Array`, `Swift.Dictionary`,
      or `Foundation.NSNull`
    - All dictionary keys are `Swift.String`s
    - `NSNumber`s are not NaN or infinity
*/

@available(*, unavailable)
extension JSONSerializationX : @unchecked Sendable { }

open class JSONSerializationX : NSObject {
    
    /* Determines whether the given object can be converted to JSON.
       Other rules may apply. Calling this method or attempting a conversion are the definitive ways
       to tell if a given object can be converted to JSON data.
       - parameter obj: The object to test.
       - returns: `true` if `obj` can be converted to JSON, otherwise `false`.
     */
    open class func isValidJSONObject(_ obj: Any) -> Bool {
        var recursionDepth = 0
        
        // TODO: - revisit this once bridging story gets fully figured out
        func isValidJSONObjectInternal(_ obj: Any?) -> Bool {
            // Match Darwin Foundation in not considering a deep object valid.
            guard recursionDepth < JSONSerializationX.maximumRecursionDepth else { return false }
            recursionDepth += 1
            defer { recursionDepth -= 1 }
            
            // Emulate the SE-0140 behavior bridging behavior for nils
            guard let obj = obj else {
                return true
            }
            
//            if !(obj is _NSNumberCastingWithoutBridging) {
              if obj is String || obj is NSNull || obj is Int || obj is Bool || obj is UInt ||
                  obj is Int8 || obj is Int16 || obj is Int32 || obj is Int64 ||
                  obj is UInt8 || obj is UInt16 || obj is UInt32 || obj is UInt64 {
                  return true
              }
//            }

            // object is a Double and is not NaN or infinity
            if let number = obj as? Double  {
                return number.isFinite
            }
            // object is a Float and is not NaN or infinity
            if let number = obj as? Float  {
                return number.isFinite
            }

            if let number = obj as? Decimal {
                return number.isFinite
            }

            // object is Swift.Array
            if let array = obj as? [Any?] {
                for element in array {
                    guard isValidJSONObjectInternal(element) else {
                        return false
                    }
                }
                return true
            }

            // object is Swift.Dictionary
            if let dictionary = obj as? [String: Any?] {
                for (_, value) in dictionary {
                    guard isValidJSONObjectInternal(value) else {
                        return false
                    }
                }
                return true
            }

            // object is NSNumber and is not NaN or infinity
            // For better performance, this (most expensive) test should be last.
//            if let number = __SwiftValue.store(obj) as? NSNumber {
//                if CFNumberIsFloatType(number._cfObject) {
//                    let dv = number.doubleValue
//                    let invalid = dv.isInfinite || dv.isNaN
//                    return !invalid
//                } else {
//                    return true
//                }
//            }

            // invalid object
            return false
        }

        // top level object must be an Swift.Array or Swift.Dictionary
        guard obj is [Any?] || obj is [String: Any?] else {
            return false
        }

        return isValidJSONObjectInternal(obj)
    }
    // MARK: - JSON-Write-encode
    /* Generate JSON data from a Foundation object. If the object will not produce valid JSON then an exception will be thrown. Setting the NSJSONWritingPrettyPrinted option will generate JSON with whitespace designed to make the output more readable. If that option is not set, the most compact possible JSON will be generated. If an error occurs, the error parameter will be set and the return value will be nil. The resulting data is a encoded in UTF-8.
     */
//    internal class func _data(withJSONObject value: Any, options opt: WritingOptions, stream: Bool) throws -> Data {
//        var jsonStr = [UInt8]()
//        
//        var writer = JSONWriterX(
//            options: opt,
//            writer: { (str: String?) in
//                if let str = str {
//                    jsonStr.append(contentsOf: str.utf8)
//                }
//            }
//        )
//        
//        if let container = value as? NSArray {
//            try writer.serializeJSON(container._bridgeToSwift())
//        } else if let container = value as? NSDictionary {
//            try writer.serializeJSON(container._bridgeToSwift())
//        } else if let container = value as? Array<Any> {
//            try writer.serializeJSON(container)
//        } else if let container = value as? Dictionary<AnyHashable, Any> {
//            try writer.serializeJSON(container)
//        } else {
//            guard opt.contains(.fragmentsAllowed) else {
//                fatalError("Top-level object was not NSArray or NSDictionary") // This is a fatal error in objective-c too (it is an NSInvalidArgumentException)
//            }
//            try writer.serializeJSON(value)
//        }
//
//        let count = jsonStr.count
//        return Data(bytes: &jsonStr, count: count)
//    }

//    open class func data(withJSONObject value: Any, options opt: WritingOptions = []) throws -> Data {
//        return try _data(withJSONObject: value, options: opt, stream: false)
//    }
    
    // MARK: - JSON-Reader-decode
    /* Create a Foundation object from JSON data. Set the NSJSONReadingAllowFragments option if the parser should allow top-level objects that are not an NSArray or NSDictionary. Setting the NSJSONReadingMutableContainers option will make the parser generate mutable NSArrays and NSDictionaries. Setting the NSJSONReadingMutableLeaves option will make the parser generate mutable NSString objects. If an error occurs during the parse, then the error parameter will be set and the result will be nil.
       The data must be in one of the 5 supported encodings listed in the JSON specification: UTF-8, UTF-16LE, UTF-16BE, UTF-32LE, UTF-32BE. The data may or may not have a BOM. The most efficient encoding to use for parsing is UTF-8, so if you have a choice in encoding the data passed to this method, use UTF-8.
     */
    open class func jsonObject(with string: String?, options opt: ReadingOptions = []) throws -> Any {
        guard let data = string?.data(using: .utf8) else {
            throw JSONError.cannotConvertInputDataToUTF8
        }
        return try jsonObject(with: data, options: opt)
    }
    open class func jsonObject(with data: Data, options opt: ReadingOptions = []) throws -> Any {
        do {
            let jsonValue = try data.withUnsafeBytes { (ptr) -> JSONValueX in
                let (encoding, advanceBy) = JSONSerializationX.detectEncoding(ptr)
                
                if encoding == .utf8 {
                    // we got utf8... happy path
                    var parser = JSONParserX(bytes: Array(ptr[advanceBy..<ptr.count]))
                    return try parser.parse()
                }
                
                guard let utf8String = String(bytes: ptr[advanceBy..<ptr.count], encoding: encoding) else {
                    throw JSONError.cannotConvertInputDataToUTF8
                }
                
                var parser = JSONParserX(bytes: Array(utf8String.utf8))
                return try parser.parse()
            }
            
            if jsonValue.isValue, !opt.contains(.fragmentsAllowed) {
                throw JSONError.singleFragmentFoundButNotAllowed
            }
            
            return try jsonValue.toObjcRepresentationX(options: opt)
        } catch let error as JSONError {
            switch error {
            case .cannotConvertInputDataToUTF8:
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Cannot convert input string to valid utf8 input."
                ])
            case .unexpectedEndOfFile:
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Unexpected end of file during JSON parse."
                ])
            case .unexpectedCharacter(_, let characterIndex):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Invalid value around character \(characterIndex)."
                ])
            case .expectedLowSurrogateUTF8SequenceAfterHighSurrogate:
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Unexpected end of file during string parse (expected low-surrogate code point but did not find one)."
                ])
            case .couldNotCreateUnicodeScalarFromUInt32:
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Unable to convert hex escape sequence (no high character) to UTF8-encoded character."
                ])
            case .unexpectedEscapedCharacter(_, _, let index):
                // we lower the failure index by one to match the darwin implementations counting
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Invalid escape sequence around character \(index - 1)."
                ])
            case .singleFragmentFoundButNotAllowed:
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "JSON text did not start with array or object and option to allow fragments not set."
                ])
            case .tooManyNestedArraysOrDictionaries(characterIndex: let characterIndex):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : "Too many nested arrays or dictionaries around character \(characterIndex + 1)."
                ])
            case .invalidHexDigitSequence(let string, index: let index):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : #"Invalid hex encoded sequence in "\#(string)" at \#(index)."#
                ])
            case .unescapedControlCharacterInString(ascii: let ascii, in: _, index: let index) where ascii == UInt8(ascii: "\\"):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : #"Invalid escape sequence around character \#(index)."#
                ])
            case .unescapedControlCharacterInString(ascii: _, in: _, index: let index):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : #"Unescaped control character around character \#(index)."#
                ])
            case .numberWithLeadingZero(index: let index):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : #"Number with leading zero around character \#(index)."#
                ])
            case .numberIsNotRepresentableInSwift(parsed: let parsed):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : #"Number \#(parsed) is not representable in Swift."#
                ])
            case .invalidUTF8Sequence(let data, characterIndex: let index):
                throw NSError(domain: NSCocoaErrorDomain, code: CocoaError.propertyListReadCorrupt.rawValue, userInfo: [
                    NSDebugDescriptionErrorKey : #"Invalid UTF-8 sequence \#(data) starting from character \#(index)."#
                ])
            }
        } catch {
            preconditionFailure("Only `JSONError` expected")
        }
        
    }
    
//#if !os(WASI)
//    /* Write JSON data into a stream. The stream should be opened and configured. The return value is the number of bytes written to the stream, or 0 on error. All other behavior of this method is the same as the dataWithJSONObject:options:error: method.
//     */
//    open class func writeJSONObject(_ obj: Any, toStream stream: OutputStream, options opt: WritingOptions) throws -> Int {
//        let jsonData = try _data(withJSONObject: obj, options: opt, stream: true)
//        return jsonData.withUnsafeBytes { (rawBuffer: UnsafeRawBufferPointer) -> Int in
//            let ptr = rawBuffer.baseAddress!.assumingMemoryBound(to: UInt8.self)
//            let res: Int = stream.write(ptr, maxLength: rawBuffer.count)
//            /// TODO: If the result here is negative the error should be obtained from the stream to propagate as a throw
//            return res
//        }
//    }
//    
//    /* Create a JSON object from JSON data stream. The stream should be opened and configured. All other behavior of this method is the same as the JSONObjectWithData:options:error: method.
//     */
//    open class func jsonObject(with stream: InputStream, options opt: ReadingOptions = []) throws -> Any {
//        var data = Data()
//        guard stream.streamStatus == .open || stream.streamStatus == .reading else {
//            fatalError("Stream is not available for reading")
//        }
//        repeat {
//            let buffer = try [UInt8](unsafeUninitializedCapacity: 1024) { buf, initializedCount in
//                let bytesRead = stream.read(buf.baseAddress!, maxLength: buf.count)
//                initializedCount = bytesRead
//                guard bytesRead >= 0 else {
//                    throw stream.streamError!
//                }
//            }
//            data.append(buffer, count: buffer.count)
//        } while stream.hasBytesAvailable
//        return try jsonObject(with: data, options: opt)
//    }
//#endif
}

//MARK: - Encoding Detection

private extension JSONSerializationX {
    /// Detect the encoding format of the NSData contents
    static func detectEncoding(_ bytes: UnsafeRawBufferPointer) -> (String.Encoding, Int) {
        // According to RFC8259, the text encoding in JSON must be UTF8 in nonclosed systems
        // https://tools.ietf.org/html/rfc8259#section-8.1
        // However, since Darwin Foundation supports utf16 and utf32, so should Swift Foundation.
        
        // First let's check if we can determine the encoding based on a leading Byte Ordering Mark
        // (BOM).
        if bytes.count >= 4 {
            if bytes.starts(with: Self.utf8BOM) {
                return (.utf8, 3)
            }
            if bytes.starts(with: Self.utf32BigEndianBOM) {
                return (.utf32BigEndian, 4)
            }
            if bytes.starts(with: Self.utf32LittleEndianBOM) {
                return (.utf32LittleEndian, 4)
            }
            if bytes.starts(with: [0xFF, 0xFE]) {
                return (.utf16LittleEndian, 2)
            }
            if bytes.starts(with: [0xFE, 0xFF]) {
                return (.utf16BigEndian, 2)
            }
        }
        
        // If there is no BOM present, we might be able to determine the encoding based on
        // occurrences of null bytes.
        if bytes.count >= 4 {
            switch (bytes[0], bytes[1], bytes[2], bytes[3]) {
            case (0, 0, 0, _):
                return (.utf32BigEndian, 0)
            case (_, 0, 0, 0):
                return (.utf32LittleEndian, 0)
            case (0, _, 0, _):
                return (.utf16BigEndian, 0)
            case (_, 0, _, 0):
                return (.utf16LittleEndian, 0)
            default:
                break
            }
        }
        else if bytes.count >= 2 {
            switch (bytes[0], bytes[1]) {
            case (0, _):
                return (.utf16BigEndian, 0)
            case (_, 0):
                return (.utf16LittleEndian, 0)
            default:
                break
            }
        }
        return (.utf8, 0)
    }
    
    static func parseBOM(_ bytes: UnsafeRawBufferPointer) -> (encoding: String.Encoding, skipLength: Int)? {
        
        return nil
    }
    
    // These static properties don't look very nice, but we need them to
    // workaround: https://bugs.swift.org/browse/SR-14102
    private static let utf8BOM: [UInt8] = [0xEF, 0xBB, 0xBF]
    private static let utf32BigEndianBOM: [UInt8] = [0x00, 0x00, 0xFE, 0xFF]
    private static let utf32LittleEndianBOM: [UInt8] = [0xFF, 0xFE, 0x00, 0x00]
    private static let utf16BigEndianBOM: [UInt8] = [0xFF, 0xFE]
    private static let utf16LittleEndianBOM: [UInt8] = [0xFE, 0xFF]
}

enum JSONValueX: Equatable {
    case string(String)
    case number(String)
    case bool(Bool)
    case null

    case array([JSONValueX])
    case object([String: JSONValueX])
}

extension JSONValueX {
    var isValue: Bool {
        switch self {
        case .array, .object:
            return false
        case .null, .number, .string, .bool:
            return true
        }
    }
    
    var isContainer: Bool {
        switch self {
        case .array, .object:
            return true
        case .null, .number, .string, .bool:
            return false
        }
    }
}

extension JSONValueX {
    var debugDataTypeDescription: String {
        switch self {
        case .array:
            return "an array"
        case .bool:
            return "bool"
        case .number:
            return "a number"
        case .string:
            return "a string"
        case .object:
            return "a dictionary"
        case .null:
            return "null"
        }
    }
}

private extension JSONValueX {
    func toObjcRepresentationX(options: JSONSerializationX.ReadingOptions) throws -> Any {
        switch self {
        case .array(let values):
            let array = try values.map { try $0.toObjcRepresentationX(options: options) }
            if !options.contains(.mutableContainers) {
                return array
            }
            return NSMutableArray(array: array, copyItems: false)
        case .object(let object):
            let dictionary = try object.mapValues { try $0.toObjcRepresentationX(options: options) }
            if !options.contains(.mutableContainers) {
                return dictionary
            }
            return NSMutableDictionary(dictionary: dictionary, copyItems: false)
        case .bool(let bool):
            return NSNumber(value: bool)
        case .number(let string):
            guard let number = NSNumber.fromJSONNumberX(string, options: options) else {
                throw JSONError.numberIsNotRepresentableInSwift(parsed: string)
            }
            return number
        case .null:
            return NSNull()
        case .string(let string):
            if options.contains(.mutableLeaves) {
                return NSMutableString(string: string)
            }
            return string
        }
    }
}

extension NSNumber {
    static func fromJSONNumberX(_ string: String, options: JSONSerializationX.ReadingOptions) -> Any? {
        let decIndex = string.firstIndex(of: ".")
        let expIndex = string.firstIndex(of: "e")
        let isInteger = decIndex == nil && expIndex == nil
        let isNegative = string.utf8[string.utf8.startIndex] == UInt8(ascii: "-")
        let digitCount = string[string.startIndex..<(expIndex ?? string.endIndex)].count
        
        // Try Int64() or UInt64() first
        if isInteger {
            if isNegative {
                if digitCount <= 19, let intValue = Int64(string) {
                    return NSNumber(value: intValue)
                }
            } else {
                if digitCount <= 20, let uintValue = UInt64(string) {
                    return NSNumber(value: uintValue)
                }
            }
        }

        var exp = 0
        
        if let expIndex = expIndex {
            let expStartIndex = string.index(after: expIndex)
            if let parsed = Int(string[expStartIndex...]) {
                exp = parsed
            }
        }
        
        // Decimal holds more digits of precision but a smaller exponent than Double
        // so try that if the exponent fits and there are more digits than Double can hold
        if digitCount > 17, exp >= -128, exp <= 127, let decimal = Decimal(string: string), decimal.isFinite {
            return NSDecimalNumber(decimal: decimal)
        }
        var str = string
        // Fall back to Double() for everything else
        if options.contains(.doubleToString), (string.contains("e") || string.contains("E")) {
            // 科学计数法的数字
            str = "\(NSDecimalNumber(string: string))"
        }
        // 如果原始数字，或科学计算后的数字是小数，返回字符串
        if options.contains(.doubleToString), str.contains(".") {
            return str
        }
        if let doubleValue = Double(string), doubleValue.isFinite {
            return NSNumber(value: doubleValue)
        }
        
        return nil
    }
}


//protocol _NSNumberCastingWithoutBridging {
//  var _swiftValueOfOptimalType: (Any & Sendable) { get }
//}
//
//extension NSNumber: _NSNumberCastingWithoutBridging {
//    internal var _swiftValueOfOptimalType: (Any & Sendable) {
////        if self === kCFBooleanTrue {
////            return true
////        } else if self === kCFBooleanFalse {
////            return false
////        }
////        
////        let numberType = _CFNumberGetType2(_cfObject)
////        switch numberType {
////        case kCFNumberSInt8Type:
////            return Int(int8Value)
////        case kCFNumberSInt16Type:
////            return Int(int16Value)
////        case kCFNumberSInt32Type:
////            return Int(int32Value)
////        case kCFNumberSInt64Type:
////            return int64Value < Int.max ? Int(int64Value) : int64Value
////        case kCFNumberFloat32Type:
////            return floatValue
////        case kCFNumberFloat64Type:
////            return doubleValue
////        case kCFNumberSInt128Type:
////            // If the high portion is 0, return just the low portion as a UInt64, which reasonably fixes trying to roundtrip UInt.max and UInt64.max.
////            if int128Value.high == 0 {
////                return int128Value.low
////            } else {
////                return int128Value
////            }
////        default:
////            fatalError("unsupported CFNumberType: '\(numberType)'")
////        }
//        return true
//    }
//}
internal struct JSONParserX {
    var reader: DocumentReader
    var depth: Int = 0

    init(bytes: [UInt8]) {
        self.reader = DocumentReader(array: bytes)
    }

    mutating func parse() throws -> JSONValueX {
        try reader.consumeWhitespace()
        let value = try self.parseValue()
        #if DEBUG
        defer {
            guard self.depth == 0 else {
                preconditionFailure("Expected to end parsing with a depth of 0")
            }
        }
        #endif

        // ensure only white space is remaining
        var whitespace = 0
        while let next = reader.peek(offset: whitespace) {
            switch next {
            case ._space, ._tab, ._return, ._newline:
                whitespace += 1
                continue
            default:
                throw JSONError.unexpectedCharacter(ascii: next, characterIndex: reader.readerIndex + whitespace)
            }
        }

        return value
    }

    // MARK: Generic Value Parsing

    mutating func parseValue() throws -> JSONValueX {
        var whitespace = 0
        while let byte = reader.peek(offset: whitespace) {
            switch byte {
            case UInt8(ascii: "\""):
                reader.moveReaderIndex(forwardBy: whitespace)
                return .string(try reader.readString())
            case ._openbrace:
                reader.moveReaderIndex(forwardBy: whitespace)
                let object = try parseObject()
                return .object(object)
            case ._openbracket:
                reader.moveReaderIndex(forwardBy: whitespace)
                let array = try parseArray()
                return .array(array)
            case UInt8(ascii: "f"), UInt8(ascii: "t"):
                reader.moveReaderIndex(forwardBy: whitespace)
                let bool = try reader.readBool()
                return .bool(bool)
            case UInt8(ascii: "n"):
                reader.moveReaderIndex(forwardBy: whitespace)
                try reader.readNull()
                return .null
            case UInt8(ascii: "-"), UInt8(ascii: "0") ... UInt8(ascii: "9"):
                reader.moveReaderIndex(forwardBy: whitespace)
                let number = try self.reader.readNumber()
                return .number(number)
            case ._space, ._return, ._newline, ._tab:
                whitespace += 1
                continue
            default:
                throw JSONError.unexpectedCharacter(ascii: byte, characterIndex: self.reader.readerIndex)
            }
        }

        throw JSONError.unexpectedEndOfFile
    }


    // MARK: - Parse Array -

    mutating func parseArray() throws -> [JSONValueX] {
        precondition(self.reader.read() == ._openbracket)
        guard self.depth < 512 else {
            throw JSONError.tooManyNestedArraysOrDictionaries(characterIndex: self.reader.readerIndex - 1)
        }
        self.depth += 1
        defer { depth -= 1 }

        // parse first value or end immediately
        switch try reader.consumeWhitespace() {
        case ._space, ._return, ._newline, ._tab:
            preconditionFailure("Expected that all white space is consumed")
        case ._closebracket:
            // if the first char after whitespace is a closing bracket, we found an empty array
            self.reader.moveReaderIndex(forwardBy: 1)
            return []
        default:
            break
        }

        var array = [JSONValueX]()
        array.reserveCapacity(10)

        // parse values
        while true {
            let value = try parseValue()
            array.append(value)

            // consume the whitespace after the value before the comma
            let ascii = try reader.consumeWhitespace()
            switch ascii {
            case ._space, ._return, ._newline, ._tab:
                preconditionFailure("Expected that all white space is consumed")
            case ._closebracket:
                reader.moveReaderIndex(forwardBy: 1)
                return array
            case ._comma:
                // consume the comma
                reader.moveReaderIndex(forwardBy: 1)
                // consume the whitespace before the next value
                if try reader.consumeWhitespace() == ._closebracket {
                    // the foundation json implementation does support trailing commas
                    reader.moveReaderIndex(forwardBy: 1)
                    return array
                }
                continue
            default:
                throw JSONError.unexpectedCharacter(ascii: ascii, characterIndex: reader.readerIndex)
            }
        }
    }

    // MARK: - Object parsing -

    mutating func parseObject() throws -> [String: JSONValueX] {
        precondition(self.reader.read() == ._openbrace)
        guard self.depth < 512 else {
            throw JSONError.tooManyNestedArraysOrDictionaries(characterIndex: self.reader.readerIndex - 1)
        }
        self.depth += 1
        defer { depth -= 1 }

        // parse first value or end immediatly
        switch try reader.consumeWhitespace() {
        case ._space, ._return, ._newline, ._tab:
            preconditionFailure("Expected that all white space is consumed")
        case ._closebrace:
            // if the first char after whitespace is a closing bracket, we found an empty array
            self.reader.moveReaderIndex(forwardBy: 1)
            return [:]
        default:
            break
        }

        var object = [String: JSONValueX]()
        object.reserveCapacity(20)

        while true {
            let key = try reader.readString()
            let colon = try reader.consumeWhitespace()
            guard colon == ._colon else {
                throw JSONError.unexpectedCharacter(ascii: colon, characterIndex: reader.readerIndex)
            }
            reader.moveReaderIndex(forwardBy: 1)
            try reader.consumeWhitespace()
            object[key] = try self.parseValue()

            let commaOrBrace = try reader.consumeWhitespace()
            switch commaOrBrace {
            case ._closebrace:
                reader.moveReaderIndex(forwardBy: 1)
                return object
            case ._comma:
                reader.moveReaderIndex(forwardBy: 1)
                if try reader.consumeWhitespace() == ._closebrace {
                    // the foundation json implementation does support trailing commas
                    reader.moveReaderIndex(forwardBy: 1)
                    return object
                }
                continue
            default:
                throw JSONError.unexpectedCharacter(ascii: commaOrBrace, characterIndex: reader.readerIndex)
            }
        }
    }
}

extension JSONParserX {

    struct DocumentReader {
        let array: [UInt8]

        private(set) var readerIndex: Int = 0

        private var readableBytes: Int {
            self.array.endIndex - self.readerIndex
        }

        var isEOF: Bool {
            self.readerIndex >= self.array.endIndex
        }


        init(array: [UInt8]) {
            self.array = array
        }

        subscript<R: RangeExpression<Int>>(bounds: R) -> ArraySlice<UInt8> {
            self.array[bounds]
        }

        mutating func read() -> UInt8? {
            guard self.readerIndex < self.array.endIndex else {
                self.readerIndex = self.array.endIndex
                return nil
            }

            defer { self.readerIndex += 1 }

            return self.array[self.readerIndex]
        }

        func peek(offset: Int = 0) -> UInt8? {
            guard self.readerIndex + offset < self.array.endIndex else {
                return nil
            }

            return self.array[self.readerIndex + offset]
        }

        mutating func moveReaderIndex(forwardBy offset: Int) {
            self.readerIndex += offset
        }

        @discardableResult
        mutating func consumeWhitespace() throws -> UInt8 {
            var whitespace = 0
            while let ascii = self.peek(offset: whitespace) {
                switch ascii {
                case ._space, ._return, ._newline, ._tab:
                    whitespace += 1
                    continue
                default:
                    self.moveReaderIndex(forwardBy: whitespace)
                    return ascii
                }
            }

            throw JSONError.unexpectedEndOfFile
        }

        mutating func readString() throws -> String {
            try self.readUTF8StringTillNextUnescapedQuote()
        }

        mutating func readNumber() throws -> String {
            try self.parseNumber()
        }

        mutating func readBool() throws -> Bool {
            switch self.read() {
            case UInt8(ascii: "t"):
                guard self.read() == UInt8(ascii: "r"),
                      self.read() == UInt8(ascii: "u"),
                      self.read() == UInt8(ascii: "e")
                else {
                    guard !self.isEOF else {
                        throw JSONError.unexpectedEndOfFile
                    }

                    throw JSONError.unexpectedCharacter(ascii: self.peek(offset: -1)!, characterIndex: self.readerIndex - 1)
                }

                return true
            case UInt8(ascii: "f"):
                guard self.read() == UInt8(ascii: "a"),
                      self.read() == UInt8(ascii: "l"),
                      self.read() == UInt8(ascii: "s"),
                      self.read() == UInt8(ascii: "e")
                else {
                    guard !self.isEOF else {
                        throw JSONError.unexpectedEndOfFile
                    }

                    throw JSONError.unexpectedCharacter(ascii: self.peek(offset: -1)!, characterIndex: self.readerIndex - 1)
                }

                return false
            default:
                preconditionFailure("Expected to have `t` or `f` as first character")
            }
        }

        mutating func readNull() throws {
            guard self.read() == UInt8(ascii: "n"),
                  self.read() == UInt8(ascii: "u"),
                  self.read() == UInt8(ascii: "l"),
                  self.read() == UInt8(ascii: "l")
            else {
                guard !self.isEOF else {
                    throw JSONError.unexpectedEndOfFile
                }

                throw JSONError.unexpectedCharacter(ascii: self.peek(offset: -1)!, characterIndex: self.readerIndex - 1)
            }
        }

        // MARK: - Private Methods -

        // MARK: String

        enum EscapedSequenceError: Swift.Error {
            case expectedLowSurrogateUTF8SequenceAfterHighSurrogate(index: Int)
            case unexpectedEscapedCharacter(ascii: UInt8, index: Int)
            case couldNotCreateUnicodeScalarFromUInt32(index: Int, unicodeScalarValue: UInt32)
        }

        private mutating func readUTF8StringTillNextUnescapedQuote() throws -> String {
            guard self.read() == ._quote else {
                throw JSONError.unexpectedCharacter(ascii: self.peek(offset: -1)!, characterIndex: self.readerIndex - 1)
            }
            var stringStartIndex = self.readerIndex
            var copy = 0
            var output: String?

            while let byte = peek(offset: copy) {
                switch byte {
                case UInt8(ascii: "\""):
                    self.moveReaderIndex(forwardBy: copy + 1)
                    guard var result = output else {
                        // if we don't have an output string we create a new string
                        return try makeString(at: stringStartIndex ..< stringStartIndex + copy)
                    }
                    // if we have an output string we append
                    result += try makeString(at: stringStartIndex ..< stringStartIndex + copy)
                    return result

                case 0 ... 31:
                    // All Unicode characters may be placed within the
                    // quotation marks, except for the characters that must be escaped:
                    // quotation mark, reverse solidus, and the control characters (U+0000
                    // through U+001F).
                    var string = output ?? ""
                    let errorIndex = self.readerIndex + copy
                    string += try makeString(at: stringStartIndex ... errorIndex)
                    throw JSONError.unescapedControlCharacterInString(ascii: byte, in: string, index: errorIndex)

                case UInt8(ascii: "\\"):
                    self.moveReaderIndex(forwardBy: copy)
                    if output != nil {
                        output! += try makeString(at: stringStartIndex ..< stringStartIndex + copy)
                    } else {
                        output = try makeString(at: stringStartIndex ..< stringStartIndex + copy)
                    }

                    let escapedStartIndex = self.readerIndex

                    do {
                        let escaped = try parseEscapeSequence()
                        output! += escaped
                        stringStartIndex = self.readerIndex
                        copy = 0
                    } catch EscapedSequenceError.unexpectedEscapedCharacter(let ascii, let failureIndex) {
                        output! += try makeString(at: escapedStartIndex ..< self.readerIndex)
                        throw JSONError.unexpectedEscapedCharacter(ascii: ascii, in: output!, index: failureIndex)
                    } catch EscapedSequenceError.expectedLowSurrogateUTF8SequenceAfterHighSurrogate(let failureIndex) {
                        output! += try makeString(at: escapedStartIndex ..< self.readerIndex)
                        throw JSONError.expectedLowSurrogateUTF8SequenceAfterHighSurrogate(in: output!, index: failureIndex)
                    } catch EscapedSequenceError.couldNotCreateUnicodeScalarFromUInt32(let failureIndex, let unicodeScalarValue) {
                        output! += try makeString(at: escapedStartIndex ..< self.readerIndex)
                        throw JSONError.couldNotCreateUnicodeScalarFromUInt32(
                            in: output!, index: failureIndex, unicodeScalarValue: unicodeScalarValue
                        )
                    }

                default:
                    copy += 1
                    continue
                }
            }

            throw JSONError.unexpectedEndOfFile
        }

        private func makeString<R: RangeExpression<Int>>(at range: R) throws -> String {
            let raw = array[range]
            guard let str = String(bytes: raw, encoding: .utf8) else {
                throw JSONError.invalidUTF8Sequence(Data(raw), characterIndex: range.relative(to: array).lowerBound)
            }
            return str
        }

        private mutating func parseEscapeSequence() throws -> String {
            precondition(self.read() == ._backslash, "Expected to have an backslash first")
            guard let ascii = self.read() else {
                throw JSONError.unexpectedEndOfFile
            }

            switch ascii {
            case 0x22: return "\""
            case 0x5C: return "\\"
            case 0x2F: return "/"
            case 0x62: return "\u{08}" // \b
            case 0x66: return "\u{0C}" // \f
            case 0x6E: return "\u{0A}" // \n
            case 0x72: return "\u{0D}" // \r
            case 0x74: return "\u{09}" // \t
            case 0x75:
                let character = try parseUnicodeSequence()
                return String(character)
            default:
                throw EscapedSequenceError.unexpectedEscapedCharacter(ascii: ascii, index: self.readerIndex - 1)
            }
        }

        private mutating func parseUnicodeSequence() throws -> Unicode.Scalar {
            // we build this for utf8 only for now.
            let bitPattern = try parseUnicodeHexSequence()

            // check if high surrogate
            let isFirstByteHighSurrogate = bitPattern & 0xFC00 // nil everything except first six bits
            if isFirstByteHighSurrogate == 0xD800 {
                // if we have a high surrogate we expect a low surrogate next
                let highSurrogateBitPattern = bitPattern
                guard let (escapeChar) = self.read(),
                      let (uChar) = self.read()
                else {
                    throw JSONError.unexpectedEndOfFile
                }

                guard escapeChar == UInt8(ascii: #"\"#), uChar == UInt8(ascii: "u") else {
                    throw EscapedSequenceError.expectedLowSurrogateUTF8SequenceAfterHighSurrogate(index: self.readerIndex - 1)
                }

                let lowSurrogateBitBattern = try parseUnicodeHexSequence()
                let isSecondByteLowSurrogate = lowSurrogateBitBattern & 0xFC00 // nil everything except first six bits
                guard isSecondByteLowSurrogate == 0xDC00 else {
                    // we are in an escaped sequence. for this reason an output string must have
                    // been initialized
                    throw EscapedSequenceError.expectedLowSurrogateUTF8SequenceAfterHighSurrogate(index: self.readerIndex - 1)
                }

                let highValue = UInt32(highSurrogateBitPattern - 0xD800) * 0x400
                let lowValue = UInt32(lowSurrogateBitBattern - 0xDC00)
                let unicodeValue = highValue + lowValue + 0x10000
                guard let unicode = Unicode.Scalar(unicodeValue) else {
                    throw EscapedSequenceError.couldNotCreateUnicodeScalarFromUInt32(
                        index: self.readerIndex, unicodeScalarValue: unicodeValue
                    )
                }
                return unicode
            }

            guard let unicode = Unicode.Scalar(bitPattern) else {
                throw EscapedSequenceError.couldNotCreateUnicodeScalarFromUInt32(
                    index: self.readerIndex, unicodeScalarValue: UInt32(bitPattern)
                )
            }
            return unicode
        }

        private mutating func parseUnicodeHexSequence() throws -> UInt16 {
            // As stated in RFC-8259 an escaped unicode character is 4 HEXDIGITs long
            // https://tools.ietf.org/html/rfc8259#section-7
            let startIndex = self.readerIndex
            guard let firstHex = self.read(),
                  let secondHex = self.read(),
                  let thirdHex = self.read(),
                  let forthHex = self.read()
            else {
                throw JSONError.unexpectedEndOfFile
            }

            guard let first = DocumentReader.hexAsciiTo4Bits(firstHex),
                  let second = DocumentReader.hexAsciiTo4Bits(secondHex),
                  let third = DocumentReader.hexAsciiTo4Bits(thirdHex),
                  let forth = DocumentReader.hexAsciiTo4Bits(forthHex)
            else {
                let hexString = String(decoding: [firstHex, secondHex, thirdHex, forthHex], as: Unicode.UTF8.self)
                throw JSONError.invalidHexDigitSequence(hexString, index: startIndex)
            }
            let firstByte = UInt16(first) << 4 | UInt16(second)
            let secondByte = UInt16(third) << 4 | UInt16(forth)

            let bitPattern = UInt16(firstByte) << 8 | UInt16(secondByte)

            return bitPattern
        }

        private static func hexAsciiTo4Bits(_ ascii: UInt8) -> UInt8? {
            switch ascii {
            case 48 ... 57:
                return ascii - 48
            case 65 ... 70:
                // uppercase letters
                return ascii - 55
            case 97 ... 102:
                // lowercase letters
                return ascii - 87
            default:
                return nil
            }
        }

        // MARK: Numbers

        private enum ControlCharacter {
            case operand
            case decimalPoint
            case exp
            case expOperator
        }

        private mutating func parseNumber() throws -> String {
            var pastControlChar: ControlCharacter = .operand
            var numbersSinceControlChar: UInt = 0
            var hasLeadingZero = false

            // parse first character

            guard let ascii = self.peek() else {
                preconditionFailure("Why was this function called, if there is no 0...9 or -")
            }
            switch ascii {
            case UInt8(ascii: "0"):
                numbersSinceControlChar = 1
                pastControlChar = .operand
                hasLeadingZero = true
            case UInt8(ascii: "1") ... UInt8(ascii: "9"):
                numbersSinceControlChar = 1
                pastControlChar = .operand
            case UInt8(ascii: "-"):
                numbersSinceControlChar = 0
                pastControlChar = .operand
            default:
                preconditionFailure("Why was this function called, if there is no 0...9 or -")
            }

            var numberchars = 1

            // parse everything else
            while let byte = self.peek(offset: numberchars) {
                switch byte {
                case UInt8(ascii: "0"):
                    if hasLeadingZero {
                        throw JSONError.numberWithLeadingZero(index: readerIndex + numberchars)
                    }
                    if numbersSinceControlChar == 0, pastControlChar == .operand {
                        // the number started with a minus. this is the leading zero.
                        hasLeadingZero = true
                    }
                    numberchars += 1
                    numbersSinceControlChar += 1
                case UInt8(ascii: "1") ... UInt8(ascii: "9"):
                    if hasLeadingZero {
                        throw JSONError.numberWithLeadingZero(index: readerIndex + numberchars)
                    }
                    numberchars += 1
                    numbersSinceControlChar += 1
                case UInt8(ascii: "."):
                    guard numbersSinceControlChar > 0, pastControlChar == .operand else {
                        throw JSONError.unexpectedCharacter(ascii: byte, characterIndex: readerIndex + numberchars)
                    }

                    numberchars += 1
                    hasLeadingZero = false
                    pastControlChar = .decimalPoint
                    numbersSinceControlChar = 0

                case UInt8(ascii: "e"), UInt8(ascii: "E"):
                    guard numbersSinceControlChar > 0,
                          pastControlChar == .operand || pastControlChar == .decimalPoint
                    else {
                        throw JSONError.unexpectedCharacter(ascii: byte, characterIndex: readerIndex + numberchars)
                    }

                    numberchars += 1
                    hasLeadingZero = false
                    pastControlChar = .exp
                    numbersSinceControlChar = 0
                case UInt8(ascii: "+"), UInt8(ascii: "-"):
                    guard numbersSinceControlChar == 0, pastControlChar == .exp else {
                        throw JSONError.unexpectedCharacter(ascii: byte, characterIndex: readerIndex + numberchars)
                    }

                    numberchars += 1
                    pastControlChar = .expOperator
                    numbersSinceControlChar = 0
                case ._space, ._return, ._newline, ._tab, ._comma, ._closebracket, ._closebrace:
                    guard numbersSinceControlChar > 0 else {
                        throw JSONError.unexpectedCharacter(ascii: byte, characterIndex: readerIndex + numberchars)
                    }
                    let numberStartIndex = self.readerIndex
                    self.moveReaderIndex(forwardBy: numberchars)

                    return String(decoding: self[numberStartIndex ..< self.readerIndex], as: Unicode.UTF8.self)
                default:
                    throw JSONError.unexpectedCharacter(ascii: byte, characterIndex: readerIndex + numberchars)
                }
            }

            guard numbersSinceControlChar > 0 else {
                throw JSONError.unexpectedEndOfFile
            }

            defer { self.readerIndex = self.array.endIndex }
            return String(decoding: self.array.suffix(from: readerIndex), as: Unicode.UTF8.self)
        }
    }
}

extension UInt8 {

    internal static let _space = UInt8(ascii: " ")
    internal static let _return = UInt8(ascii: "\r")
    internal static let _newline = UInt8(ascii: "\n")
    internal static let _tab = UInt8(ascii: "\t")

    internal static let _colon = UInt8(ascii: ":")
    internal static let _comma = UInt8(ascii: ",")

    internal static let _openbrace = UInt8(ascii: "{")
    internal static let _closebrace = UInt8(ascii: "}")

    internal static let _openbracket = UInt8(ascii: "[")
    internal static let _closebracket = UInt8(ascii: "]")

    internal static let _quote = UInt8(ascii: "\"")
    internal static let _backslash = UInt8(ascii: "\\")

}

extension Array where Element == UInt8 {

    internal static let _true = [UInt8(ascii: "t"), UInt8(ascii: "r"), UInt8(ascii: "u"), UInt8(ascii: "e")]
    internal static let _false = [UInt8(ascii: "f"), UInt8(ascii: "a"), UInt8(ascii: "l"), UInt8(ascii: "s"), UInt8(ascii: "e")]
    internal static let _null = [UInt8(ascii: "n"), UInt8(ascii: "u"), UInt8(ascii: "l"), UInt8(ascii: "l")]

}

enum JSONError: Swift.Error, Equatable {
    case cannotConvertInputDataToUTF8
    case unexpectedCharacter(ascii: UInt8, characterIndex: Int)
    case unexpectedEndOfFile
    case tooManyNestedArraysOrDictionaries(characterIndex: Int)
    case invalidHexDigitSequence(String, index: Int)
    case unexpectedEscapedCharacter(ascii: UInt8, in: String, index: Int)
    case unescapedControlCharacterInString(ascii: UInt8, in: String, index: Int)
    case expectedLowSurrogateUTF8SequenceAfterHighSurrogate(in: String, index: Int)
    case couldNotCreateUnicodeScalarFromUInt32(in: String, index: Int, unicodeScalarValue: UInt32)
    case numberWithLeadingZero(index: Int)
    case numberIsNotRepresentableInSwift(parsed: String)
    case singleFragmentFoundButNotAllowed
    case invalidUTF8Sequence(Data, characterIndex: Int)
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

do {
    if let data = str.data(using: .utf8) {
      let dict5 = try JSONSerializationX.jsonObject(with: data, options: [.doubleToString])
      let name5 = dict5?.getString("name")
      let age5 = dict5?.getInt32("age")
      let height5 = dict5?.getString("height")
      print(dict5 ?? [:])
      print("name5: \(name5 ?? "") age5: \(age5 ?? 0) height5: \(height5 ?? "")\n")
    }
} catch {
    print(error)
}

do {
    if let data = str.data(using: .utf8) {
      let dict6 = try JSONSerialization.jsonObject(with: data, options: [])
      let name6 = dict6?.getString("name")
      let age6 = dict6?.getInt32("age")
      let height6 = dict6?.getString("height")
      print(dict6 ?? [:])
      print("name6: \(name6 ?? "") age6: \(age6 ?? 0) height6: \(height6 ?? "")\n")
    }
} catch {
    print(error)
}

```

#### 性能说明：
json字符串如果很短，性能差异不大
如果是比较长的json字符串，差异十分明显
```
解析同一个长json字符串，每个个库耗时差异好明显,官方出品，果然精品
startDate: 2025-03-11 10:42:53.792 -- swiftdo/json
endDate: 2025-03-11 10:42:54.468 dis:675-- swiftdo/json 

startDate: 2025-03-11 10:42:54.468 -- swiftlang/swift-corelibs-foundation
endDate: 2025-03-11 10:42:54.476 dis:7-- swiftlang/swift-corelibs-foundation 

startDate: 2025-03-11 10:42:54.476 -- foundation/JSONSerialization
endDate: 2025-03-11 10:42:54.476 dis:1-- foundation/JSONSerialization

JSONSerialization > JSONSerializationX > JSONParser
```
![image](/assets/img/other/jsonDot.png)