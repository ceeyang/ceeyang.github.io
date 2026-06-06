---
title: Swift Debug Print
date: 2019-08-10
tags:
    - iOS
    - Swift
description: Swift Debug Print
---

## 自定义 swift 控制台输出

```swift

// MARK: - Debug Print -
public func print<T>(_ message: T, fileName: String = #file, methodName: String = #function, line: Int = #line) {
    #if DEBUG
    let printInfo =
    """
    \n
    =================================================================================================================================================
    【File    Name】: \(fileName.split("/").last ??  "")
    【FunctionName】: \(methodName)
    【Fuction Line】: \(line)
    【Print   Time】: \(Date())
    【   Message  】:
    \(message)

    =================================================================================================================================================
    \n
    """
    print(printInfo, separator: "", terminator: "")
    #endif
}

```
