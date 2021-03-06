---
layout: article
title: 信息存储理论与技术
key: storage-theory
tags: Storage CourseNotes
---

信息存储理论与技术课程笔记

<!-- more -->

__多层次研究__

* 器件
* 设备
* 系统

<br/>

* 存储 Storage
* 存取 Access

__存取方法__

* 顺序
* 直接：块间直接到达、块内顺序存取
* 随机
* 关联

__存储系统关键特性及性能指标__

* 存储位置：CPU、内存、外存
* 容量
* 传送单位：字（主存）、块（外存）

* 性能
    * 存取时间

        读写操作时间、定位时间

    * 存储周期（随机存储器）

        存取时间 + 附加时间

    * 传送速度
        * 随机存储器：$$1 / 周期$$
        * 非随机存储器：$$读写N位时间 \ T_N = 平均存取时间 \ T_A + \frac{N}{R}$$
    * 集合带宽
    * 访问延迟
    * 吞吐率

__存储级别__

1. 在线存储 On-line storage
2. 近线存储（联机后备存储） Near-line storage
3. 离线存储（脱机后备存储） Off-line storage

__存储系统发展__

1. 直接附加存储 Direct Attached Storage, DAS 盘阵
2. 附网存储 Network-attached Storage, NAS 文件系统
3. 存储区域网 Storage Area Network, SAN 存储池

__基于对象存储系统__

三方：Metadata Server (MDS), client, OSD
