---
layout: post
title: "procfs与sysfs"
category: kernel
date: 2025-02-22 15:00:00 +0800
---

## 介绍

procfs提供进程、内核运行时信息获取，sysctl参数配置等功能，除`/proc/sys`用于配置sysctl参数外，接口通常时只读的。

sysfs在2.6版本引入，对外提供内核各子系统、硬件、驱动的配置接口。

## 接口开发

Todo

## 常用的procfs、sysfs接口

|接口路径|作用|
|-|-|
|/proc/sys/|内核参数的读写路径|
|/proc/iomem|物理地址布局，内核实际可用的物理内存是System RAM，free查到的结果需要从System RAM中删除一些Reserved等特殊用途的物理内存。|
|/proc/meminfo|内存数据统计：匿名页、文件页、大页、交换分区等等大小 <https://blog.39hope.com/?p=184>|
|/proc/cpuinfo|CPU详情：核心、厂商、支持的功能等|
|/proc/buddyinfo|伙伴系统的详细信息|
|/proc/slabinfo|slab的详细信息|
|/proc/interrupts|检查中断触发统计情况|
|/proc/softirqs|检查软中断触发统计情况|
|/proc/PID/maps|进程虚拟地址布局|
|/proc/PID/smaps|类似/proc/PID/maps，提供更多细节：<https://www.cnblogs.com/aspirs/p/13896571.html>|
|/proc/sys/kernel/sched_domain|CPU的调度域|
|--------------------|--------------------|
|/sys/kernel/mm|内存子系统的常用配置：hugepages、swap、transparent_hugepage|
|/sys/devices/system/cpu|CPU的详细信息，拓扑结构|