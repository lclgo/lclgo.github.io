---
layout:       post
title:        "rust中Result快速进行值转换的语法糖"
author:       "licunlong"
header-style: text
catalog:      true
tags:
    - rust
---

Rust可以快速使用map、map_or等一大类的语法糖进行快速的`Result<T, E>`的值转换。但是这些东西在我看来很容易混淆，专门记录一下：

### map_or_else

输入：提供两个匿名函数，同时转换T、E。
输出：看匿名函数，可以不输出返回值。

### map_err

输入：提供一个匿名函数，转换E。
输出：Result

### map_or

输入：提供一个匿名函数，转换T。提供一个缺省值应对E。
输出：匿名函数和缺省值必须是相同的类型。

### or_else

输入：提供一个匿名函数，转换E。
输出：Result
这个和map_err有些类似。
