---
title: "dart_common_extensions：一套 Dart 常用扩展方法集合"
date: 2026-06-03
tags: ["Dart", "Flutter", "Open Source", "Package"]
description: "开源了一套 Dart 扩展方法库：方便日常开发中处理日期格式化、字符串校验、列表操作、Map 变换等常见场景，减少样板代码。"
---

## 前言

写 Dart / Flutter 时，你一定遇到过这些场景：

> 字符串转 int 要写 `int.parse(s)`，还担心抛异常；日期格式化到处写 `DateFormat`；列表想要个 `chunked` 还得自己写循环；Map 想取个不存在的 key 要加一大堆 `containsKey` 判断……

这些小操作本身不难，但每天写十几次，代码就变得啰嗦且容易出错。

这就是我开源 **dart_common_extensions** 的原因。

项目地址：[github.com/ceeyang/dart_common_extensions](https://github.com/ceeyang/dart_common_extensions)  
Pub 地址：[pub.dev/packages/dart_common_extensions](https://pub.dev/packages/dart_common_extensions)

```
⭐ 150+ 扩展方法，覆盖 String / DateTime / List / Map / Num / Enum / Object
📦 零运行时依赖（仅 decimal + intl）
🧪 100+ 单元测试
📄 完整 API 文档 + 示例代码
```

---

## 安装

```yaml
dependencies:
  dart_common_extensions: ^0.0.9
```

```bash
# Flutter 项目
flutter pub get

# 纯 Dart 项目
dart pub get
```

---

## 核心功能一览

### 📎 Object Extensions — 函数式编程小工具

Kotlin 开发者会非常熟悉 `let` 和 `also`，Dart 里也能用了：

```dart
// let：对非空对象执行操作并返回结果
final length = userInput?.let((it) => it.length) ?? 0;

// also：对对象执行副作用并返回对象本身
someObject.also((it) => print('Processing: $it'));

// 空安全判断
print(someObject.isNull);    // true / false
print(someObject.isNotNull); // true / false
```

### 🧵 String Extensions — 字符串处理的瑞士军刀

这是这个包里**最常用的部分**。类型转换、校验、格式化、大小写变换，一行搞定：

**类型转换：**

```dart
'12'.toInt;                          // 12
'12.34'.toDouble;                    // 12.34
'2023-01-01'.toDate;                 // DateTime(2023, 1, 1)
'{"key":"value"}'.toJson;            // {key: value}
'1,2,3'.toIntList;                   // [1, 2, 3]
```

**校验方法：**

```dart
'13912345678'.isChineseMobile;       // true
'user@example.com'.isEmail;          // true
'192.168.1.1'.isIP;                  // true
'{"a":1}'.isJson;                    // true
'SGVsbG8='.isBase64;                 // true
'https://example.com'.isValidUrl;    // true（支持 localhost、IP、自定义端口）
'AFFE'.isValidHex;                   // true
```

**大小写转换：**

```dart
'hello_world'.toCamelCase;           // helloWorld
'helloWorld'.toSnakeCase;            // hello_world
'hello-world'.toKebabCase;           // hello-world
'hello world'.toTitleCase;           // Hello World
```

**字符串操作：**

```dart
'This is a long text'.truncate(10);  // 'This is a ...'
'user@example.com'.substringBefore('@'); // 'user'
'hello'.capitalize;                  // 'Hello'
'Hello World'.removeAllSpaces;       // 'HelloWorld'
```

### 📅 DateTime Extensions — 日期时间不头疼

**格式化：**

```dart
final now = DateTime.now();

now.ymd;              // "2026-07-15"
now.dmy;              // "15-07-2026"
now.iso8601;          // "2026-07-15T12:00:00"
now.fullDateTime;     // "2026-07-15 12:00:00"
now.monthYear;        // "July 2026"
now.shortDate;        // "7/15/2026"（locale dependent）
now.longDate;         // "July 15, 2026"（locale dependent）
now.time;             // "12:00:00"
```

字符串也可以直接格式化：

```dart
'2023-01-01'.ymd;                    // "2023-01-01"
'2023-01-01'.fullDateTime;           // "2023-01-01 00:00:00"
'1672531200000'.ymd;                 // "2023-01-01"（时间戳也支持）
```

**日期计算：**

```dart
now.startOfDay;                      // 当天 00:00:00.000
now.endOfDay;                        // 当天 23:59:59.999
now.isToday;                         // true / false
now.nextDay;                         // 明天
now.isLeapYear;                      // 是否闰年
now.daysInMonth;                     // 当月天数

// 工作日计算
DateTime friday = DateTime(2023, 1, 6);
friday.addBusinessDays(1);           // 下周一（跳过周末）
friday.subtractBusinessDays(1);      // 上周四
DateTime(2023, 1, 7).isWeekend;      // true（周六）
```

### 📋 List & Iterable Extensions — 集合操作大补全

**安全访问：**

```dart
['a', 'b', 'c'].safeElementAt(5);    // null（不抛异常）
[1, 2, 3, 4, 5].firstWhereOrNull((e) => e > 10); // null
```

**分组与变换：**

```dart
[1, 2, 3, 4, 5].chunked(2);         // [[1, 2], [3, 4], [5]]
[1, 2, 3, 4].windowed(2);           // [[1, 2], [2, 3], [3, 4]]
[1, 2].zip(['a', 'b']);             // [[1, 'a'], [2, 'b']]
[1, 2, 3].mapIndexed((i, e) => '$i: $e'); // ['0: 1', '1: 2', '2: 3']
```

**统计聚合：**

```dart
[1, 2, 3, 4].sum;                   // 10
[1, 2, 3, 4].average;               // 2.5
[1, 2, 3, 4].max;                   // 4
[1, 2, 3, 4].min;                   // 1
[1, 2, 3, 4].count((i) => i > 2);  // 2
```

**实用方法：**

```dart
[1, 2, 3].random;                   // 随机元素
[1, 2, 3].shuffled;                 // 打乱后的新列表
[1, 2, 3].groupBy((e) => e.isEven ? 'even' : 'odd');
// {'odd': [1, 3], 'even': [2]}
[null, null].isAllNull;             // true
```

### 🗺️ Map Extensions — 安全又灵活

```dart
var map = {'first': 1, 'second': 2};

map.getOrDefault('third', 0);       // 0（不抛异常）
map.getOrNull('third');             // null
map.toJsonString();                 // '{"first": 1, "second": 2}'

// 过滤
map.pick(['first']);                // {'first': 1}
map.omit(['first']);                // {'second': 2}
map.filterKeys((k) => k.startsWith('f')); // {'first': 1}
map.filterValues((v) => v > 1);     // {'second': 2}

// 合并
{'a': 1}.merge({'a': 2, 'b': 3});  // {'a': 2, 'b': 3}
{'a': 1}.merge({'a': 2}, (v1, v2) => v1 + v2); // {'a': 3}

// 变换
{'a': 1}.mapKeys((k, v) => k.toUpperCase()); // {'A': 1}
{'a': 1}.mapValues((k, v) => v + 1);         // {'a': 2}
```

### 🔢 Num Extensions — 数字也可以有语法糖

**时间 Duration：**

```dart
5.seconds;            // Duration(seconds: 5)
1.5.days;             // Duration(hours: 36)
60.secondsDuration;   // 0:01:00.000000
await 5.secondsDelay(); // 延迟 5 秒
```

**格式化：**

```dart
1024.toFileSize();              // "1.00 KB"
123456.78.toCurrency(symbol: '€'); // "€123,456.78"
1234.567.toMoney(decimalDigits: 1); // "1,234.6"
255.toHexString;                // "ff"
10.toBinaryString;              // "1010"
0.1234.toPercentage();          // "12.34%"
```

**数学：**

```dart
1.rangeTo(5);                   // [1, 2, 3, 4, 5]
5.rangeTo(1);                   // [5, 4, 3, 2, 1]
1.sumTo(5);                     // 15
2.power(3);                     // 8
5.isPrime;                      // true
2.isEven;                       // true
```

### 🔁 Enum Extensions — 循环导航

```dart
enum Status { active, inactive }

Status.active.next(Status.values);    // Status.inactive
Status.inactive.previous(Status.values); // Status.active
```

特别适合分页状态机、步骤导航等场景。

---

## 为什么自己造这个轮子？

Dart 生态中其实不缺扩展库，比如 `dartx`、`basics`、`supercharged` 等。选择自建一个，有几个原因：

1. **按需定制**：只收录日常真的会用到的扩展，不做大而全的"百宝箱"
2. **中文友好**：内置了 `isChinese`、`isChineseMobile` 等中文场景校验
3. **简单透明**：代码量不大，每个扩展方法都清晰可查，遇到问题直接看源码比翻文档快
4. **零学习成本**：所有方法命名都是自解释的，看一眼就知道干嘛用的

---

## 项目状态与路线图

当前版本 `0.0.9`，2026年5月底初次发布，已有 150+ 扩展方法和 100+ 单元测试。

后续计划：

- [x] `Num` 扩展：货币格式化、范围生成、延迟
- [x] `List` 扩展：`windowed`、`zip`、`groupBy`
- [x] `String` 扩展：百分比、Base64、URL 编码
- [ ] **Iterable** 扩展补充：更多懒加载操作
- [ ] **Duration** 扩展：人类可读格式
- [ ] **Null** 扩展：`isNull` / `isNotNull` 顶层函数
- [ ] **更多本地化** 日期格式

欢迎提 Issue 或 PR 贡献你常用的扩展方法。

---

## 结语

`dart_common_extensions` 不是什么惊天动地的项目，它是一个**工具箱**——把每天写 Dart / Flutter 时那些"又来一遍"的操作封装起来，让代码更短、更清晰、更安全。

项目是完全开源的（MIT 协议），欢迎 Star、Issue 和 PR：

👉 [github.com/ceeyang/dart_common_extensions](https://github.com/ceeyang/dart_common_extensions)

如果你也在写 Dart / Flutter，不妨试一试，加一行依赖就能省下不少样板代码。
