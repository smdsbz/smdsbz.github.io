---
layout: article
title: Edge Storage Research
key: edge-storage-research
tags: Storage EdgeComputing EdgeStorage
---

Research note for a new project. The term _edge storage_ may differ from what
you'd expect.

<!-- more -->

Defining Edge Storage
=====================

What it usually mean?
---------------------

* [边缘存储](https://blog.csdn.net/Owen_Liangcheng/article/details/103274598)
* [星际云通](https://www.szxjyt.cn/xjyt/storage/)

__一般定义与应用场景__

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjExODgxMS0zZmM5NmQxZWUzODk0MDkxP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvNjAwL2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

边缘存储（Edge Storage）就是把数据直接存储在数据采集点，而不需要把采集的数据通过网络（即时）传输到存储的数据中心服务器（或云存储）的数据存储方式。例如：
* 公共监控摄像：在摄像头本地保存数据，即时处理（[eg](https://www.axis.com/en-us/technologies/edge-storage#:~:text=Edge%20storage%20is%20a%20technology%20used%20in%20Axis,the%20cost%20and%20effort%20for%20remote%20site%20recording.)）
* 家庭数据中心：用户希望数据存储在自己家中，而不是存储与安全防护的公司，希望公司不接触数据，兼顾隐私与安全
* 车联网采集数据：往往可以在端进行先期处理，把整理后的少量数据传给数据中心

| ![星际云通](https://www.szxjyt.cn/static/images/bianbanner2.png) |
|:-:|
| 星际云通总体架构 |

__边缘计算的好处__

* 网络带宽的有效利用
* 部署将更加容易
* 容错性更强
* 安全与隐私兼顾
* 与边缘计算结合

__基于边缘存储的点对点网络__

网络加速，利用边缘存储建立分发网络。由于设备非常分散，分发加速的效果将远远好于当前站点有限的 CDN
网络。

当边缘存储进入实用阶段，去中心化的应用也更容易建立。由于基于边缘存储的点对点网络的建立，使得应用或服务商之间的数据共享变得更加容易和便捷：服务商或应用提供商完全可以不拥有数据，数据本身属于数据的生产者，数据的拥有者可以选择把数据分享给不同的应用或服务。

TODO:

In our context
--------------

__Intuition__

网络分块（Partition）更加明显的数据中心。

分块内的网络与分块之间的网络有明显的性能异构性，甚至可能在接入方式上也异构。

TODO:

-------------------------------------------

Ceph with Partitioned Network
=============================

TODO:
