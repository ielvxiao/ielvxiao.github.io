---
title: JVM中的MetaspaceSize与MaxMetaspaceSize参数
description: "JVM中的MetaspaceSize与MaxMetaspaceSize参数"
locale: "zh_CN"
author: Xiao Lv
date: 2022-06-30
category: Jekyll
layout: post
---
# 遇到问题
应用启动后就触发Full GC，很奇怪。后续通过查询GC日志初步判断是因为Metaspace扩容导致的Full GC。
# 什么是MeataSpace
Google了MetaSpace的概念如下
> Metaspace is a native memory region that stores metadata for classes. As a class is loaded by the JVM, its metadata (i.e. its runtime representation in the JVM) is allocated into the Metaspace. The Metaspace occupancy grows as more and more classes are loaded.Metaspace is a native memory region that stores metadata for classes. As a class is loaded by the JVM, its metadata (i.e. its runtime representation in the JVM) is allocated into the Metaspace.  
The Metaspace occupancy grows as more and more classes are loaded. And, when a classloader and all its loaded classes become unreachable in the Java heap, the associated class metadata in the Metaspace becomes eligible for deallocation. The class metadata is cleaned up with a garbage collection cycle.

大概意思就是：
1. Metaspace属于本地堆内存(native heap)。
2. 作用是用来存储class metadata。
3. Metaspace也会触发GC。
# MetaSpace控制
JVM里有一些参数用来控制MetaspaceSize
- -XX:MetaspaceSize：Metaspace 空间初始大小，如果不设置的话，默认是20.79M，`这个初始大小是触发首次 Metaspace Full GC 的阈值`，例如 -XX:MetaspaceSize=256M

- -XX:MaxMetaspaceSize：Metaspace 最大值，默认不限制大小，但是线上环境建议设置，例如
-XX:MaxMetaspaceSize=256M

- -XX:MinMetaspaceFreeRatio：最小空闲比，当 Metaspace 发生 GC 后，会计算 Metaspace 的空闲比，如果空闲比(空闲空间/当前 Metaspace 大小)小于此值，就会触发 Metaspace 扩容。默认值是 40 ，也就是 40%，例如 -XX:MinMetaspaceFreeRatio=40

- -XX:MaxMetaspaceFreeRatio:最大空闲比，当 Metaspace 发生 GC 后，会计算 Metaspace 的空闲比，如果空闲比(空闲空间/当前 Metaspace 大小)大于此值，就会触发 Metaspace 释放空间。默认值是 70 ，也就是 70%，例如 -XX:MaxMetaspaceFreeRatio=70
# 问题解决
通过调大MaxMetaspaceSize进行测试.
```
-XX:MetaspaceSize=256m 
-XX:MaxMetaspaceSize=256m
```
发现问题已经解决，应用启动后不再Full GC。
# 引用
>https://poonamparhar.github.io/understanding-metaspace-gc-logs
> https://stackoverflow.com/questions/36465192/guidelines-to-set-metaspacesize-java-8
