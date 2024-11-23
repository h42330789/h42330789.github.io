---
title: swift里extension+where区别使用状态代替
author: 独孤流
date: 2024-11-23 01:04:00 +0800
categories: [Tool]
tags: [Tool,Swift]     # TAG names should always be lowercase
---

----
> ### 前言
> 组里大佬分享了一个利用`swift`的`Extension+where`进行类型代替状态的代码设计思路，将在代码运行期间才会发现的错提前利用编译工具在编译期间就能提示，基本原理是指定泛型才有特定的方法，每一步都转换生成一个新的对象

使用场景：定义一个类型人，人的状态分男性、女性、其他，不同年龄段有不同的方法\
方式一，直接使用枚举
```
struct People {
    enum GenderType {
        case male
        case female
    }

    var gender: GenderType
    var name: String
    var age: Int
    init(gender: GenderType, name: String, age: Int) {
        self.gender = gender
        self.name = name
        self.age = age
    }

    func beWife() {
        switch gender {
        case .male:
            // 只有女性才能成为妻子，这个只有在运行期间才知道有问题
            print("are you sure?")
            break
        case .female:
            print("good")
            break
        }
    }
    // 只有男性才能成为丈夫
    func beHusband() {
        switch gender {
        case .male:
            // 只有男性才能成为丈夫，这个只有在运行期间才知道有问题
            print("good")
            break
        case .female:
            print("are you sure?")
            break
        }
    }
    
    mutating func changeGenger(gender: GenderType) {
        self.gender = gender
    }
}
People(gender: .female, name: "aa", age: 18).beWife() // 正常
People(gender: .female, name: "bb", age: 18).beHusband() // 编译期间正常，运行期间才会提示错误
// 如果后期再增加其他性别，所有判断的地方都要修改，比较麻烦
```

方式二、使用子类的方式实现，由于`struct`不能继承，需要定义`class`
```
class PeopleModel {
    enum GenderType {
        case male
        case female
    }
    
    var gender: GenderType
    var name: String
    var age: Int
    init(gender: GenderType, name: String, age: Int) {
        self.gender = gender
        self.name = name
        self.age = age
    }
    
}
class FemalePeopleModel: PeopleModel {
    
    convenience init(name: String, age: Int) {
        self.init(gender: .female, name: name, age: age)
    }
    func beWife() {
      print("good")
    }
    
    func changeMale() -> MalePeopleModel {
        return MalePeopleModel(name: name, age: age)
    }
}
class MalePeopleModel: PeopleModel {
    convenience init(name: String, age: Int) {
        self.init(gender: .male, name: name, age: age)
    }
    func changeFemale() -> FemalePeopleModel {
        return FemalePeopleModel(name: name, age: age)
    }
    func beHusband() {
        print("good")
    }
}

FemalePeopleModel(name: "aa", age: 18).beWife() // 正常
MalePeopleModel(name: "bb", age: 18).beHusband() // 正常
MalePeopleModel(name: "cc", age: 18).changeFemale().beWife()
MalePeopleModel(name: "dd", age: 18).beWife() // 编译期间就会提示错误
```

方式三：使用Swift的Extension+Where
```
enum PeoleGender {
    enum Female {}
    enum Male {}
}
struct PeopleData<T> {
    
    var name: String
    var age: Int
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

}

extension PeopleData where T == PeoleGender.Female {
    func beWife() {
        print("good")
    }
    func changeMale() -> PeopleData<PeoleGender.Male> {
        return PeopleData<PeoleGender.Male>(name: name, age: age)
    }
}
extension PeopleData where T == PeoleGender.Male {
    func beHusband() {
        print("good")
    }
    func changeFemale() -> PeopleData<PeoleGender.Female> {
        return PeopleData<PeoleGender.Female>(name: name, age: age)
    }
}
PeopleData<PeoleGender.Female>( name: "aa", age: 18).beWife() // 正常
PeopleData<PeoleGender.Male>(name: "bb", age: 18).beHusband() // 正常
PeopleData<PeoleGender.Male>(name: "cc", age: 18).changeFemale().beWife() // 正常
PeopleData<PeoleGender.Male>(name: "dd", age: 18).beWife() // 编译期间会提示错误
```

使用泛型代替属性代表状态及限制方法使用
```
enum UserGradePrimary {} // 小学
enum UserGradeMiddle {} // 中学
enum UserGradeHigh {} // 中学

struct UserModel<T> {
    var name: String
    var age: Int
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}
extension UserModel where T == UserGradePrimary {
    @discardableResult
    func studyMath() -> UserModel<UserGradePrimary> {
        // ....
        return self
    }
    func update2Middle() -> UserModel<UserGradeMiddle> {
        return UserModel<UserGradeMiddle>(name: name, age: age)
    }
}
extension UserModel where T == UserGradeMiddle {
    @discardableResult
    func studyEnglish() -> UserModel<UserGradeMiddle> {
        // ...
        return self
    }
    func update2High() -> UserModel<UserGradeHigh> {
        return UserModel<UserGradeHigh>(name: name, age: age)
    }
}
UserModel<UserGradePrimary>(name: "aa", age: 10)
    .studyMath()
    .update2Middle()
    .studyEnglish()
    .update2High()

// 本例子中，泛型有2个功能
// 功能1：本身就代表着一种用户状态信息代替定义一个变量来记录信息
// 功能2：泛型约束响应类型只能调用特定的方法，如果方法调错，在编译期间直接报错

// 这种模式在实际的业务开发中使用很少，因为业务开发基本遵循短平快，且需求一开始基本都很简单，要这么设计可能反而更费时，只是随着需求的变化变得复杂会使用上，但是这个时候一般人都秉持着重构太费事，而且风险比较高，干脆在旧有的基础上稍微改改完成功能就行

// 这种模式的适用场景是业务比较固定，在写基础框架或者底层时，由于代码调用比较广泛，需要限制比较严格，代码不想随意暴露给上层，这种模式就可以既限制代码，还能让上层能增加适配实现
```

----
> ### 前言二
> swift里有很多方便的语法，尤其配合上where条件，使用起来非常便利

#### 1. 用于 for 循环
在 `for-in` 循环中，`where` 可以用来为循环添加过滤条件。

```
let numbers = [1, 2, 3, 4, 5, 6]

// 只打印偶数
for number in numbers where number % 2 == 0 {
    print(number)
}
// 输出:
// 2
// 4
// 6
```

#### 2. 用于 `switch` 语句
`where` 可在 `case` 分支中添加额外的条件。

```
let point = (3, 0)

switch point {
case let (x, y) where x == y:
    print("点位于 y = x 上")
case let (x, y) where x == -y:
    print("点位于 y = -x 上")
case let (x, y) where x > 0 && y == 0:
    print("点在 x 轴的正方向上")
default:
    print("点在其他地方")
}
// 输出:
// 点在 x 轴的正方向上

```

#### 3. 用于泛型约束
`where` 可以用来为泛型添加额外的条件，通常与协议一起使用。

```
func allItemsMatch<T: Sequence, U: Sequence>(_ first: T, _ second: U) -> Bool
    where T.Element == U.Element, T.Element: Equatable {
    for (item1, item2) in zip(first, second) {
        if item1 != item2 {
            return false
        }
    }
    return true
}

let array1 = [1, 2, 3]
let array2 = [1, 2, 3]

print(allItemsMatch(array1, array2)) // 输出: true

```

#### 4. 用于扩展
在扩展中，`where` 可用来限定扩展的适用范围。

```
extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        guard let first = self.first else { return true }
        return allSatisfy { $0 == first }
    }
}

let array = [1, 1, 1, 1]
print(array.allEqual()) // 输出: true

enum AnimalType {
    enum Dog {}
    enum Cat {}
    enum Panda {}
}
struct Animal<T> {
    private(set) var name: String
}
extension Animal where T == AnimalType.Dog {
    func wangwang() {
        
    }
}
extension Animal where T == AnimalType.Cat {
    func miao() {
        
    }
}
```

#### 5. 用于协议关联类型
在协议中，`where` 可以用来对关联类型进行约束。

```
protocol Container {
    associatedtype Item
    func contains(_ item: Item) -> Bool
}

extension Container where Item: Equatable {
    func contains(_ item: Item) -> Bool {
        // 实现包含检查逻辑
        return true
    }
}

```