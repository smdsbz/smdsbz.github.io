---
layout: article
title: Edge Storage Research
key: edge-storage-research
tags: Storage DistributedStorage EdgeComputing EdgeStorage EdgeDataCenter
---

Research note for a new project. The meaning of term _Edge Storage_ may differ
from what you'd expect.

<!-- more -->

Concepts
========

Defining Edge Storage
---------------------

### What it usually mean?

* [边缘存储](https://blog.csdn.net/Owen_Liangcheng/article/details/103274598)
* [星际云通](https://www.szxjyt.cn/xjyt/storage/)
* [边缘计算：分布式存储的另一种可能](https://zhuanlan.zhihu.com/p/166374976)
* __[边缘计算：跨越传统数据中心](https://amotoki.github.io/edge-computing-whitepaper/zh_CN/)__
    * original whitepaper on [OpenStack](https://www.openstack.org/use-cases/edge-computing/)
* [Open Glossary of Edge Computing](https://github.com/State-of-the-Edge/glossary/blob/master/edge-glossary.md)

#### 一般定义与应用场景

边缘计算是为应用开发者和服务提供商在网络的边缘侧提供云服务和 IT 环境服务，目标是在靠近数据输入或用户的地方提供计算、存储和网络带宽。边缘计算着重解决的问题是传统云计算（或中央计算）模式下存在的高延迟、网络不稳定和低带宽的问题。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjExODgxMS0zZmM5NmQxZWUzODk0MDkxP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvNjAwL2Zvcm1hdC93ZWJw?x-oss-process=image/format,png)

边缘存储（Edge Storage）就是把数据直接存储在数据采集点，而不需要把采集的数据通过网络（即时）传输到存储的数据中心服务器（或云存储）的数据存储方式。例如：
* 公共监控摄像：在摄像头本地保存数据，即时处理（[Axis](https://www.axis.com/en-us/technologies/edge-storage#:~:text=Edge%20storage%20is%20a%20technology%20used%20in%20Axis,the%20cost%20and%20effort%20for%20remote%20site%20recording.)）
* 家庭数据中心：用户希望数据存储在自己家中，而不是存储与安全防护的公司，希望公司不接触数据，兼顾隐私与安全
* 车联网采集数据：往往可以在端进行先期处理，把整理后的少量数据传给数据中心

| ![星际云通](https://www.szxjyt.cn/static/images/bianbanner2.png) |
|:-:|
| 星际云通总体架构 |

边缘计算适用于与物联网设备紧密相关、数据传输延迟敏感、数据交互次数多、数据传输量大的物联网应用；而云存储则适用于延迟敏感度稍低的互联网应用。

#### 边缘计算的好处

* 网络带宽的有效利用
    * 降低延迟
    * 消除网络依赖
* 部署将更加容易
* 容错性更强
* 安全与隐私兼顾
* 与边缘计算结合
    * 数据实时性、时效性

#### 边缘计算环境的特点

* 多个站点之间的潜在高延迟
* 网络不可达、慢速带宽
* 伴随一般数据中心中心化资源池所不能应对的其他交付服务和应用功能

#### 边缘计算的能力

* 提供一个跨多种基础设施的一致性操作范式
* 能够支持大规模分布式环境
* 能够为全球分布的客户交付网络服务
* 能够满足应用集成、编排和服务交付的需求
* 能够满足硬件资源的限制和成本的限制
* 能够运行在局限及不稳定的网络之上
* 能够满足应用对超低延迟的需求
* 能够实现区域隔离，保护本地数据的隐私

#### 边缘计算的部署/管理挑战

* 需要标准化的和统一的基础环境。每个区域具有类似架构、已知数量
* 提供自动化管理；在处理部署、更替任何可复原性故障时提供简洁直接的（simple and
    straightforward）处理方法
* 当硬件出现故障时，提供简洁高效的（simple, cost-effective）应对计划
* 本地式容错设计也不容忽视，尤其是面对远程或不可达环境，零接触式的管理模式
* 维护操作必须简洁。未经过培训的工人能够进行人工修复和替换，而熟练的远程管理员可以进行重装或软件维护
* 物理设计可能需要整体反思。大多数的边缘计算环境并不理想，有限的能源、灰尘、湿度及震动都应该被考虑在内

![](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/5GEdge.svg)

| ![](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/edge/whitepaper/centraldatacenter.jpg) |
|:-:|
| A detailed view of the edge data center with an automated system used to operate a shrimp farm |

#### 边缘计算模型

##### 中心式控制平面 Centralized Control Plane

![](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/edge/whitepaper/centraldatacenter2.jpg)

Edge infrastructures is built as traditional single data center environment
which is geographically distributed with WAN connections between the controller
and compute nodes.

Compute services incoporate running bare metal, containerized and virtualized
workloads alike.

While the management and orchestration services are centralized, this
architecture is less resilient to failures from network connection loss. The
edge data center does not have full automony.

In summary, this architecture model does not fulfill every use case, but it
provides an evolution path to already existing architectures. Plus, it also suits
the needs of scenarios where autonomous behavior is not a requirement.

##### 分布式控制平面 Distributed Control Plane

![](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/edge/whitepaper/centraldatacenter3.jpg)

The majority of the control services redie on the large/medium edge data centers.
This provides an orchestrational overhead to synchronize between these data
centers and manage them individually and as part of a large, connected environment
at the same time.

<!--
There are different options that can be used to overcome the operational
challenges of this model. One method is to use federation techniques to connect
the databases to operate the infrastructure as a whole; another option is to
synchronize the databases across sites to make sure they have the same working
set of configurations across the deployment.
-->

| ![](https://docs.starlingx.io/_images/starlingx-deployment-options-distributed-cloud.png) |
|:-:|
| [StarlingX Distributed Cloud](https://docs.starlingx.io/deploy_install_guides/r5_release/distributed_cloud/index.html) |

#### 基于边缘存储的点对点网络

网络加速，利用边缘存储建立分发网络。由于设备非常分散，分发加速的效果将远远好于当前站点有限的 CDN
网络。

当边缘存储进入实用阶段，去中心化的应用也更容易建立。由于基于边缘存储的点对点网络的建立，使得应用或服务商之间的数据共享变得更加容易和便捷：服务商或应用提供商完全可以不拥有数据，数据本身属于数据的生产者，数据的拥有者可以选择把数据分享给不同的应用或服务。

### In our context

#### Intuition

网络分块（Partition）更加明显的数据中心。

分块内的网络与分块之间的网络有明显的性能异构性，甚至可能在接入方式上也异构。

#### 边缘数据中心

在项目的上下文中，_边缘存储_ 更多是指 _边缘数据中心_ 上的分布式存储。

* [边缘数据中心规划发展研究](https://zhuanlan.zhihu.com/p/172160917)
* [多站融合边缘数据中心方案 - 新华三集团](https://www.h3c.com/cn/d_202012/1362371_473305_0.htm#)

边缘数据中心在靠近用户的网络边缘提供基础设施资源，支持边缘计算对本地化、实时性的数据进行分析、处理、执行以及反馈，从而对云计算能力进行补充。借助边缘计算，数据可以在边缘数据中心进行处理，从而节省到传统集中式数据中心的通信等待时间。

__分类思路__

![分类思路](https://pic4.zhimg.com/80/v2-bec5bb04e2c2dbb17d0a51b25bf69723_720w.jpg)

* 电力规划与接入
* 应用场景
* 下沉位置：近端 vs. 近云

__规划与设计__

靠近用户、物理分散、逻辑统一

![规划与设计](https://pic4.zhimg.com/80/v2-f7919af3749c38bdbd302ca9926cc1f7_720w.jpg)

__发展方向__

* 模块化
* 预制产品化
    * 游牧数据中心（Nomadic Data Center）  
        Nawab F, Agrawal D, El Abbadi A. Nomadic datacenter sat the network edge: Data management challenges for the cloud with mobile infrastructure[C]. EDBT, 2018:497-500.
* 标准化

> 新华三边缘数据中心机房采用微模块标准化建设模式，提供标准接口、部件、子系统、整体架构全方位标准化、模块化定义数据中心，实现客户按需定制，非常适合多站融合边缘计算场景的云基础设施快速部署。

| ![](https://www.h3c.com/cn/res/202012/09/20201209_5411526_image002_1362371_473305_0.jpg) |
|:-:|
| 新华三云边协同处理流程 |

-------------------------------------------

Implementation and Deployment
=============================

Ceph at the Edge
----------------

### [Use Mars 400 Ceph Storage in Edge Datacenter](https://www.ambedded.com.tw/en/use-case/Use-Mars-400-Ceph-Storage-in-Edge-Datacenter/use-case-01.html)

After a series of local data processing, the result and original datasets are
stored in the edge data center. The application only uploads precise result back
to the central datacenter.

![](https://www.ambedded.com.tw/Templates/pic/Use-Case_Use-Mars-400-Ceph-for-Edge-datacenter-and-IOT.jpg)

Key reason this IoT application leads to use of Ceph:
1. Scalability with high availability
2. A storage system with unified storage protocal
3. A software-defined storage sytem can start from a mini-cluster

* [Ceph Storage Appliance - Mars 400](https://www.ambedded.com.tw/en/product/Ceph-Storage-Appliance/ceph-storage-appliance.html)
    * [Mars 400 Ceph Appliance Brochure](https://www.ambedded.com.tw/Templates/att/Ceph_Storage_Appliance_Mars_200_400_brochure.pdf?lng=en)

Every ARM server node provides dedicated CPU, memory, storage and network interface
resources to its supported object storage device (OSD). _As a result, OSD
performance is far more balanced than traditional single-server node designs
supporting many OSDs. Also, by limiting the failure domain to a single disk, the
Mars exhibits faster recovery from microserver failures._

### [OpenStack and Ceph for Distributed Hyperconverged Edge Deployments](https://thenewstack.io/openstack-and-ceph-for-distributed-hyperconverged-edge-deployments/)

> ... The resultant architecture will support NFV (which is backbone technology
> for 5G), emerging use cases with fewer control planes and distribute VNFs
> (Virtual Network Functions or network services) within all regional and edge
> nodes involved in network.

| ![](https://cdn.thenewstack.io/media/2019/02/2c730448-sn1.png) |
|:-:|
| Proposed solution referred Akraino Edge Stack (Software stack for Edge, Linux Foundation Open Source Project) |

* Ceph

    | ![](https://cdn.thenewstack.io/media/2019/02/e0647566-sn4.png) |
    |:-:|
    | Ceph for the proposed architecture |

    | ![](https://cdn.thenewstack.io/media/2019/02/5ee215ba-sn5.png) |
    |:-:|
    | Distributed Compute Nodes with Ceph |

    | ![](https://cdn.thenewstack.io/media/2019/02/6804f747-sn6.png) |
    |:-:|
    | Final Architecture showing OpenStack projects + Ceph Cluster in HCI Way |

### [What is MEC? The telco Edge. - Canonical](https://ubuntu.com/blog/what-is-mec-the-telco-edge)

| ![](https://res.cloudinary.com/canonical/image/fetch/f_auto,q_auto,fl_sanitize,c_fill,w_1440/https://lh4.googleusercontent.com/2XmTTYnmcITxV2pDUOP6abW4pPjkhgrS1HbV7CdBWX4ZeUY27vxD3wIJTXF4qbDccBnBcMYozzxY3XB47WNjgLVtLOcwv7jAhfrY8tAf9E4OP7CYFw42_he_r-AaxFxXzk6PBrhF) |
|:-:|
| Typical edge site design |

* [MAAS](https://maas.io/) to manage bare metal hardware
* [LXD](https://linuxcontainers.org/) clustering to provide an abstract layer of
    virtualization
* Ceph for distributed storage
* MicroK8s to provide a Kubernetes cluster

### [OpenShift Container Storage: Why Ceph?](https://www.openshift.com/blog/openshift-container-storage-why-ceph)

> What makes Ceph unique - ... - is that it's safe.


TODO:

Ceph with Partitioned Network
-----------------------------

TODO:
