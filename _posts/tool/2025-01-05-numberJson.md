---
title: JSON数据里number数据精度及小数位问题
author: 独孤流
date: 2025-01-05 01:04:00 +0800
categories: [Tool]
tags: [Tool,Swift]     # TAG names should always be lowercase
---

最近在做一些需求时，由于涉及到小数位计算与展示问题，跟后端协商是想让类似社交到小数位的数据都以字符串的形式返回，这样APP端在不需要计算只展示时直接展示不处理，在需要做计算时直接将字符串转换成`NSDecimalNumber`后在进行计算，这样就不至于把接口数据变成`Double`丢失精度、小数位不好控制、计算不准确问题。整个方向和实现没问题，但是后端接口做这个字段转换是使用`注解`的方式实现，可能对于`Java`来说就是`BigDecimal`,在使用`Swagger`生成的接口文档里还是`number`类型，然后一般在接口对接时，APP会把文档上的`number`类型定义成`Double`，导致这样后端即使处理好了小数位，当变成Double后要保留的小数位又不知道了，这个问题扯了好久，后端的人不知道是对技术相关的一知半解还是什么原因，说小数位可以处理，但是没法把类型变成字符串，改造这个成本太大,由于任务紧急也不会修改这个了，搞不动修改下统一的注解而已，为啥这么难，既然这个问题后端不协助修改，只能APP端自己想办法了。

由于接口返回的数据，不管定义的是什么类型，终究还是一堆字符串，于是就想到将做如下几点来适配：
1、自动生成模型的脚步，遇到`swagger`生成的类型为`number`的都统一生成类型为`String`
2、在接口的json数据赋值时，坐下兼容处理，只要key是对的，要把类型为`String、Int64、Int、Int32、Double`等基本类型都要自动转换为`String`
3、在收到接口返回的原始json的字符串时，将字符串里的带有小数点数据自动添加双引号，改造成字符串，配合按字符串获取就解决该问题

```
class TestJson {
    static func test() {
        // 模拟请求接口后获取到的数据
        let apiBodyStr = "{\"name\":\"abc\",\"age\":18,\"height\":200.12,\"nums\":[1,3,5], \"scores\":[7.8,20,-8.09,10,3.1456,\"A\",D,-90],\"web\":\"aa.com\",\"books\":[{\"pages\":99,\"price\":19.89, \"site\":\"bb.cm\"},{\"pages\":999,\"price\":100.012,\"site\":\"bb.net\"}]}"
        let formatStr = convertFloatsToStrings(in: apiBodyStr)
        print(apiBodyStr)
        print("\n")
        print(formatStr ?? "")
        print("")
    }
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
}

```
输入输出结果
```
{"name":"abc","age":18,"height":200.12,"nums":[1,3,5], "scores":[7.8,20,-8.09,10,3.1456,"A",D,-90],"web":"aa.com","books":[{"pages":99,"price":19.89, "site":"bb.cm"},{"pages":999,"price":100.012,"site":"bb.net"}]}


{"name":"abc","age":18,"height":"200.12","nums":[1,3,5], "scores":["7.8",20,"-8.09",10,"3.1456","A",D,-90],"web":"aa.com","books":[{"pages":99,"price":"19.89", "site":"bb.cm"},{"pages":999,"price":"100.012","site":"bb.net"}]}
```
