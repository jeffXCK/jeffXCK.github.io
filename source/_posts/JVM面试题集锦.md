---
title: JVM面试题集锦
comments: false
date: 2019-09-29 09:58:22
categories: JVM
tags: JVM
---

#### 1、StackOverflowError 和 OutOfMemoryError,谈谈你的理解

#### 2、一般什么时候会发生GC？如何处理？

> 答：java中的GC会有两种回收：年轻代的Minor GC ，另外一种就是养老区的Full GC；新对象创建时如果伊甸园空间不足会触发Minor GC，如果此时养老区的内存空间不足会触发Full GC，如果空间都不足则抛出OutOfMemoryError。

#### 3、GC回收策略，谈谈你的理解

> 答：
> - 新生区（伊甸园区 + 两个辛存区），GC回收策略为“复制”；
> - 养老区的保存空间一般比较大，GC回收策略为“整理-压缩”；