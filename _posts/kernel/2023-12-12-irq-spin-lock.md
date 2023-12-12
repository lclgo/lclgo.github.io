---
layout: post
title: "中断和自旋锁"
category: kernel
date: 2023-12-12 21:00:00 +0800
---

## 第一个版本

<http://arthurchiao.art/blog/linux-irq-softirq-zh/>

<https://abcdxyzk.github.io/blog/2021/06/08/base-spin-lock/>

## 自旋锁

自旋锁用于处理器之间的互斥，适合保护很短的临界区，并且不允许在临界区睡眠。申请自旋锁的时候，如果自旋锁被其他处理器占有，本处理器自旋等待（也称为忙等待，CPU可以调用PAUSE指令省电）。

进程、软中断和硬中断都可以使用自旋锁。

### 申请函数

```c
void spin_lock(spinlock_t *lock);       // 申请自旋锁，如果锁被其他处理器占有，当前处理器自旋等待。
void spin_lock_bh(spinlock_t *lock);    // 申请自旋锁，并且禁止当前处理器的软中断。
void spin_lock_irq(spinlock_t *lock);   // 申请自旋锁，并且禁止当前处理器的硬中断。
spin_lock_irqsave(lock, flags);         // 申请自旋锁，保存当前处理器的硬中断状态，并且禁止当前处理器的硬中断。
int spin_trylock(spinlock_t *lock);     //申请自旋锁，如果申请成功，返回1；如果锁被其他处理器占有，当前处理器不等待，立即返回0。
```

从这里的定义来看的话，如果处于软中断中申请自旋锁应该使用spin_lock_bh。

## 软中断

中断的相应必须及时，在中断处理期间又必须要屏蔽掉其他的中断。那么对于中断处理函数比较耗时的场景怎么处理呢？

答案是分成两部分，确实需要在中断处理函数中处理的作为上半部分，下半部分（例如网络的解包）可以放在ksoftirqd内核线程中执行。ksoftirqd每个CPU核心存在一个，si占比表示当前软中断占用的CPU，如果过高会导致用户态的进程很卡。
