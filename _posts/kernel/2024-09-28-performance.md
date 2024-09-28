---
layout: post
title: "性能分析和调优"
category: kernel, GNU-linux
date: 2024-09-28 15:00:00 +0800
---

## 参考书

<https://weedge.github.io/perf-book-cn/zh/>

## perf

### 常用参数

* `-a`：所有CPU
* `-C`：指定CPU
* `-p`：指定PID
* `-o`：指定输出文件
* `-g`：使能调用栈，支持调用关系图
* `-F`：采样频率
* `-e`：事件
* `--filter`：event时间过滤

<https://www.cnblogs.com/arnoldlu/p/6241297.html>

### 常用命令

#### 热点函数

* `perf record -g -p PID -- sleep 20`
* `perf top -p PID -g -d 100`（-d表示多久刷新一次数据）

#### 调度、时延

* `perf sched record`（通过sched record采集实验数据，然后需要在使用`perf sched latency`处理数据）

需要注意，使用`porf sched record`采集的数据必须用`perf sched`子命令来解析，而不是使用`perf report`。

#### 事件统计

* `perf stat -e dTLB-loads,dTLB-load-misses,iTLB-loads,iTLB-load-misses ./test`
* `perf stat -p 1 -e cache-reference -e cache-misses`

#### 微架构数据（TopDown）

`perf state -p 1 -ddd -- sleep 60`

* IPC：每个cycle的Instruction数量（越大越好，最好能达到甚至超过1）
* context-switches：进程切换次数（让出cpu给其他进程使用）
* cpu-migrations：当前进程在不同的CPU之间迁移的次数

### 数据解析

* `perf report --fields=overhead,symbol,dso --column-widths=0,40,0 -i perf.data`
* 火焰图：`perf script > out.perf`
* `perf annotate`：显示源代码的性能相关注释，帮助解决性能问题。

## stress-ng

待补充

## 可用于性能分析的内核接口

* `/proc/sched_debug`：详细的调度统计数据，可视化的
* `/proc/schedstat`：一连串数字，可以看到调度域信息
* `/proc/sys/kernel/sched_schedstats`：是否启用kernel debug，启用后可能会有性能下降

## 参数调优

|参数名|作用|使用场景|
|-|-|-|
|vm.dirty_writeback_centisecs|配置定期回写的间隔<br>0：禁用定期回写<br>|平衡内存使用和IO性能。对内存紧张的，使用低回写间隔，提高内存利用率；对IO性能差的使用高回写间隔，避免回写成为性能瓶颈|
|vm.swappiness|回收匿名页的激进程度（a），越高越倾向于回收匿名页，取值范围：[0,100]<br>匿名页:文件页=a:(200-a)<br>0：禁用匿名页回收<br>|对数据库这种自己对内存做管理的业务，最好将vm.swappiness配置为0，不让内核回收匿名页|
|vm.dirty_background_ratio|脏页数量达到这里设置的百分比时，开始后台脏页回写。|-|
