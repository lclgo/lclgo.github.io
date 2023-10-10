---
layout:       post
title:        "rust中Result快速进行值转换的语法糖"
author:       "licunlong"
header-style: text
catalog:      true
tags:
    - rust
---

Rust可以快速使用map、map_or等一大类的语法糖进行快速的`Result<T, E>`、`Option<V>`的值转换。但是这些东西在我看来很容易混淆，专门记录一下：

## 基本的原则

* or: 重点在于会提供一个缺省值给Err()
* or_else：重点在于提供一个匿名函数给Err()调用

### map_or_else

输入：提供两个匿名函数，同时转换T、E。
输出：看匿名函数，可以不输出返回值。

### map_err

输入：提供一个匿名函数，转换E。
输出：Result

### map_or

输入：提供一个匿名函数，转换T。提供一个缺省值应对E。
输出：匿名函数和缺省值必须是相同的类型。

### or

输入：提供一个缺省值应对E。
输出：将E转换为对应的缺省值。

### or_else

输入：提供一个匿名函数，转换E为T或E。
输出：Result
和map_err类似，区别是：map_err的匿名函数只能将E转换为E，而or_else可以将E转换为T。

### unwrap_or_else

输入：提供一个匿名函数，转换E。
输出：对于Ok，直接调用unwrap()；否则使用匿名函数做转换。

### ok_or_else（只能对Option转换）

记忆：Option<V> => Result<T, E>
输入：提供一个匿名函数，转换None。
输出：将Some转换成Ok返回；None使用匿名函数转换为E。
