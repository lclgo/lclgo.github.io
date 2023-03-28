---
layout:       post
title:        "rust的基础知识"
author:       "licunlong"
header-style: text
catalog:      true
tags:
    - rust
---

## trait

<https://zhuanlan.zhihu.com/p/127365605> trait的用法

## 智能指针

### Box

智能指针，指向堆上的数据。通过实现`Deref`来解引用`Box`。

### 计数智能指针和多重所有权

Rc<T>实现引用计数，确保这个智能指针可以在多个方法中使用。

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

### RefCell

RefCell在运行时检查借用规则，而不是在编译时。内部可变性。

## 这么写更好看?

```rust
// 如果关注返回值,后面会用到:
let fd = match socket::socket(self.sock_addr.family(), self.sa_type, falgs, self.protocol) {
    Ok(fd) => fd,
    Err(why) => {
        println!("......");
        return Err(why); 
    }
};

// 如果不关注返回值:
if let Err(why) = socket::setsockopt(fd, ReuseAddr, &true) {
    println!("......");
    return Err(why);
}
```

## 异步编程

### 1. await()的理解

调用await(),意思是等待当前的操作执行完后,再执行下一步操作.但是在等的过程中,当前是可以执行其它任务的.
如果调用thread::sleep()等待,程序会阻塞,不能执行其它任务.

### 2. 如何同时运行多个future?

join!, try_join!: 在future结束后,集中处理结果.try_join!会在单个future报错时及时中止.
select!: 

### 3. block_on会非阻塞地等待当前任务处理完成

### 4. 帮助理解asynchonous runtime

[rust-async-runtime](https://www.ncameron.org/blog/what-is-an-async-runtime/)


## 错误码的处理

```rs
#[derive(Debug, thiserror::Error)]
pub enum MyError {
    #[error("ioerror")]
    IOError(#[from] std::io::Error),
    #[error("others")]
    Others,
}

fn return_std_io_error() -> Result<(), std::io::Error> {
    Err(std::io::Error::new(std::io::ErrorKind::InvalidData, "crossesdevices"))
}

fn return_myerror() -> Result<(), MyError> {
    // if let Err(e) = return_std_io_error() {
    //     return Err(e.into());
    // }
    // Ok(())
    return_std_io_error()?;
    Ok(())
}

fn main() {
    if let Err(e) = return_myerror() {
        println!("res: {}", e.to_string());
    }
}
```

借用thiserror简化Error的处理。return_myerror函数中的`?`可以自动进行Error的类型转换，`Ok(())`继续向下走。等价于注视后的代码。

另外注意点：IOError(#[from] std::io::Error)实现了From,那么就不能重复为Others(#[from] std::io::Error)。编译器已经想到了，不允许相同的来源转换成不同的结果。

```rs
use std::error::Error;

#[derive(Debug)]
pub enum MyError {
    IOError(std::io::Error),
    Others,
}

impl std::fmt::Display for MyError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        let res = match self {
            Self::IOError(_) => "ioerror",
            Self::Others => "others",
        };
        write!(f, "{res}")
    }
}

impl Error for MyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            Self::IOError(e) => Some(e),
            Self::Others => None,
        }
    }
}

fn return_std_io_error() -> Result<(), std::io::Error> {
    Err(std::io::Error::new(std::io::ErrorKind::InvalidData, "crossesdevices"))
}

fn return_myerror() -> Result<(), MyError> {
    if let Err(e) = return_std_io_error() {
        return Err(MyError::IOError(e));
    }
    Ok(())
}

fn main() {
    if let Err(e) = return_myerror() {
        println!("res: {}", e.to_string());
        println!("cause: {}", e.source().unwrap().to_string());
    }
}
```

`impl Error for MyError`只需要实现`Debug`，`std::fmt::Display`即可。Error提供了`source`函数，能够通过这个函数将一些报错跨越抽象层传递上来。
