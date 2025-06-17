---
title: Rust里Trait及数组
author: 独孤流
date: 2025-06-17 10:14:00 +0800
categories: [rust]
tags: [iOS]     # TAG names should always be lowercase
---

由于`rust`没有继承，所以经常需要各种类似`swift`里的`protocol`的`trait`进行动态数据管理\
比如协议是`trait xxx`\
内容是`struct yyy impl xxx for yyy {}`

如果`vec!<xxx>`是不可以的，因为是协议，不是具体的类
如果`vec!<yyy>`如果yyy是具体的`enum`或`struct`是可以的

如果是协议的动态内容，必须使用
`vec!<&dyn xxx>`或者`vec!<Box<dyn xxx>>`

要允许`dyn`修饰，trait里的每个方法都要有`&self`作为第一个参数，也就是只允许出现成员方法，不允许出现类方法

由于 Rust 没有继承机制，我们使用 trait（相当于 Swift 的 protocol）实现行为抽象。使用 trait 的集合或映射时，需要特别注意：

❌ 错误用法（不能编译）
```
let v1: Vec<Table> = ...;              // ❌ trait 是不完整类型（unsized），不能直接作为元素类型
let v2: Vec<dyn Table> = ...;          // ❌ 同上，dyn Trait 也是 unsized，必须加 Box 或引用
let m1: HashMap<String, Table> = ...;  // ❌ 不合法
let m2: HashMap<String, dyn Table> = ...; // ❌ 不合法
```
✅ 正确用法
```
// ✅ 正确用法一：具体类型（静态分发）
let tables: Vec<UserTable> = vec![UserTable { ... }, ...];
let map: HashMap<String, UserTable> = HashMap::from([
    ("key1".to_string(), UserTable { ... }),
]);

// ✅ 正确用法二：引用 &dyn Trait
// 📌 生命周期由外部变量决定，不能跨线程，也不能在临时作用域后使用。
let t1 = UserTable { ... };
let t2 = OrderTable { ... };
let tables: Vec<&dyn Table> = vec![&t1, &t2];
let map: HashMap<String, &dyn Table> = HashMap::from([
    ("t1".to_string(), &t1),
    ("t2".to_string(), &t2),
]);

// ✅ 正确用法三：Box<dyn Trait>（拥有权 + 多态）
let tables: Vec<Box<dyn Table>> = vec![
    Box::new(UserTable { ... }),
    Box::new(OrderTable { ... }),
];

let map: HashMap<String, Box<dyn Table>> = HashMap::from([
    ("t1".to_string(), Box::new(UserTable { ... })),
    ("t2".to_string(), Box::new(OrderTable { ... })),
]);
```

```
trait Table {
    fn name(&self) -> &str;
}

struct UserTable {
    pub name: String
};
impl Table for UserTable {
    fn name(&self) -> &str {
        self.name
    }
}

struct OrderTable {
    pub name: String
}
impl Table for OrderTable {
    fn name(&self) -> &str {
        self.name
    }
}



// 错误
// Table 是一个 trait，不是一个具体类型，不能被用作 Vec<Table>。
let tables1: Vec<Table> = vec![
    UserTable{name: "t1".to_string()},
    OrderTable{name: "t2".to_string()},
];
let map1: HashMap<String, Table> = HashMap::from([
    ("t1", UserTable{name: "t1".to_string()}),
    ("t2", OrderTable{name: "t2".to_string()}),
]);

// 错误
// dyn Table 是 trait 对象，但不能直接存储在 Vec 里，因为它是unsized 类型。
// Rust 不允许 Vec<dyn Trait>，应使用 Box<dyn Trait> 或 &dyn Trait。
let tables2: Vec<dyn Table> = vec![
    UserTable{name: "t1".to_string()},
    OrderTable{name: "t2".to_string()},
];
let map2: HashMap<String, dyn Table> = HashMap::from([
    ("t1", UserTable{name: "t1".to_string()}),
    ("t2", OrderTable{name: "t2".to_string()}),
]);

// 正确，类型完全一致，无 trait 参与。
let tables3: Vec<UserTable> = vec![
    UserTable{name: "t1".to_string()}
    UserTable{name: "t2".to_string()}
];
let map3: HashMap<String, UserTable> = HashMap::from([
    ("t1", UserTable{name: "t1".to_string()}),
    ("t2", UserTable{name: "t2".to_string()}),
]);

// 正确，类型完全一致，无 trait 参与。
let tables4: Vec<OrderTable> = vec![
    OrderTable{name: "t1".to_string()}
    OrderTable{name: "t2".to_string()}
];
let map4: HashMap<String, OrderTable> = HashMap::from([
    ("t1", OrderTable{name: "t1".to_string()}),
    ("t2", OrderTable{name: "t2".to_string()}),
]);

// 正确
// 但是要确保对象的生命周期在vec使用期间有效
// 使用借用而不是拥有（无所有权，作用域外不能用）。
let t1 = UserTable{name: "t1".to_string()};
let t2 = OrderTable{name: "t2".to_string()}
let tables5: Vec<&dyn Table> = vec![
    &t1,
    &t2,
];
let map5: HashMap<String, &dyn Table> = HashMap::from([
    ("t1", &t1),
    ("t2", &t2),
]);

// 正确
// 最场景的trait的写法
let tables6: Vec<Box<dyn Table>> = vec![
    Box::new( UserTable{name: "t1".to_string()}),
    Box::new(OrderTable{name: "t2".to_string()}),
];
let map6: HashMap<String, Box<dyn Table>> = HashMap::from([
    ("t1",  Box::new( UserTable{name: "t1".to_string()})),
    ("t2", Box::new(OrderTable{name: "t2".to_string()})),
]);
```