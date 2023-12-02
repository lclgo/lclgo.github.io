---
layout: post
title: "kernel代码阅读入门"
category: kernel
date: 2023-06-15 08:00:00 +0800
---

## 怎么查看系统调用？

以fork为例，搜索：SYSCALL_DEFINE.*(fork)

## 零零散散

* ttwu：try_to_wake_up

## 内核态抢占

定义：在CPU执行内核代码的时候，允许调度器获得控制权。

解决的问题：

* 某些驱动程序可能陷入死循环，使系统崩溃。
* 某些驱动程序或系统调用可能执行很慢，无法将CPU归还给调度器，导致其他程序被阻塞。

## 进程优先级的高低

<http://www.wowotech.net/process_management/process-priority.html>

