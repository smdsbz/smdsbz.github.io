---
layout: article
title: 计算机系统设计
key: computer-system-design-notes
tags: ComputerSystem CourseNotes
---

计算机系统设计课程笔记

<!-- more -->

__计算机各部件功能__

* 数据处理
* 数据存储：暂存、保存
* 数据传送
* 控制：管理计算机资源，并协调各个功能部件的操作

计算发展对存储系统的性能需求上升，导致外部设备从附属设备角色向相对独立大规模分布式系统转换。

__提升系统性能的手段__

* Scale-up：提升设备性能
    * 趋近物理极限
    * 新原理与非冯诺依曼体系结构
* Scale-out：分布式
    * 能耗问题
    * 可靠性问题

-----------------------------------------------------------

内存储器件性能改进及面向外部 I/O 的 Cache 设计问题
==========================================

__问题__

* 存、算发展速度不匹配
    * 引入更复杂、更有效的高速缓存结构
    * 主存内部加入缓存（或其他缓冲机制）改进主存接口
        * EDRAM、CDRAM
    * 使主存接口“更宽” / 高速总线、分层总线缓冲、结构化数据
        * SDRAM、DDR SDRAM、堆叠内存
    * 新一代存储器件
        * 非易失：Memristor, PCM, STT-RAM, Optane PM
* 传统系统多层架构 Memory Gap
    * 容量、扩展性、能耗

__缓存预取、预寻__

* 使用线索（Hint）进行预取
    * 主动预取 Aggressive Prefetching
    * 灵通预取 Informed Prefetching
* AI 预取

__面向 I/O 的性能改进方法__



* 缓冲、暂存
* 高速总线
* I/O 处理器

面向外部 I/O（磁盘）的性能优化
===================

* 磁盘本身改进
    * 减少寻道时间：减轻移动臂质量、小盘径、提高道密度/位密度、磁阻磁头
    * 减少旋转延时、提高转速：液压轴承马达，油膜代替滚珠，避免直接摩擦，减少震动
    * 增大传输率：设备内多读/写头
* 磁盘请求调度策略
* 高速缓冲 Cache 和预取（Prefetch）技术
* 存储设备并行

可靠性
=====

__系统寿命周期__

浴盆曲线 bathtub curve

1. 早期故障期 / 调试期 Early-failure period
2. 随机故障期 Useful-life period
3. 损耗失效期 Wearout period

__系统故障率__

* 串联系统故障率：系统 MTTF = 各部件 MTTF 调和平均数
* 并联系统故障率

<!-- 能耗问题
====== -->

------------------------------------------------------

对象存储系统
=========

generalized data container

* 元数据解耦
    * 元数据与数据规模相差大
* 主动对象存储
    * 对象存储方法

__尾延迟 Tail latency__

* 降低性能波动：数据迁移、网络等
    * 与具体的系统和场景耦合度较高
* 优化资源配置：资源供给、负载均衡等
    * 用于应对系统或负载长期性的变化
* 优化请求调度：动态调度、冗余请求等
    * 用于应对系统或负载短期性的变化

存储服务质量
==========

虚拟化环境
--------

Virtualization deals with extending or replacing an existing interface so as to
mimic the behavior of another system.

* isolation
* packaging
* compatibility

SLO 精确保障
