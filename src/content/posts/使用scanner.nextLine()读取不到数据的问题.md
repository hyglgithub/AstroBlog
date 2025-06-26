---
title: 使用scanner.nextLine()读取不到数据的问题
description: 探讨 Java 中 Scanner 类的两种常用方法：nextLine() 和 next()，以及它们的区别和常见问题。
published: 2024-12-07
tags: [java]
category: 原创
draft: false
---

### `nextLine()` 与 `next()` 区别

**`nextLine()`：**
- **读取整行**，包括空格，直到遇到换行符 (`\n`)。
- 适合读取包含空格的句子或整行文本。

**`next()`：**
- **读取下一个词**，遇到空格、制表符或换行符结束。
- 适合读取单个词或连续字符序列，不包含空格。

**典型问题：**
使用 `nextLine()` 和其他 `Scanner` 方法交替使用时，可能导致读取问题：

```java
Scanner scanner = new Scanner(System.in);
System.out.print("输入一个数字：");
int num = scanner.nextInt();  // 读取数字，不消耗换行符

System.out.print("输入一行文本：");
String line = scanner.nextLine();  // 读取残留的换行符，导致输入被跳过
```

**解决方案：**
在调用 `nextLine()` 之前，添加一个额外的 `nextLine()` 清除换行符：

```java
scanner.nextLine();  // 清除残留的换行符
String line = scanner.nextLine();
```
