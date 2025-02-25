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
### 方案二、不适用系统的`JSONSerialization.jsonObject`,自己写解析JSON字符串的类,遇到数字时，如果有小数点，都转换成字符串，这样会更灵活
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

使用方式
```
let str = "{\"name\":\"jim\",\"age\":10, \"height\": 1.650, \"payList\": [{\"fee:\": -40.56},{\"fee:\": -44.00},{\"fee:\": -44.40},{\"fee:\": -42.00},{\"fee:\": 0.00}]}"

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

```

![image](/assets/img/other/jsonDot.png)