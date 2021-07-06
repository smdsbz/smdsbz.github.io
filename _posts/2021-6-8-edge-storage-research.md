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

Defining _"Edge"_
-----------------

### What it usually mean?

* [边缘存储](https://blog.csdn.net/Owen_Liangcheng/article/details/103274598)
* [星际云通](https://www.szxjyt.cn/xjyt/storage/)
* [边缘计算：分布式存储的另一种可能](https://zhuanlan.zhihu.com/p/166374976)
* __[边缘计算：跨越传统数据中心](https://amotoki.github.io/edge-computing-whitepaper/zh_CN/)__
    * original whitepaper on [OpenStack](https://www.openstack.org/use-cases/edge-computing/)
* [Open Glossary of Edge Computing](https://github.com/State-of-the-Edge/glossary/blob/master/edge-glossary.md)

#### 一般定义与应用场景

边缘计算是为应用开发者和服务提供商在网络的边缘侧提供云服务和 IT 环境服务，目标是在靠近数据输入或用户的地方提供计算、存储和网络带宽。边缘计算着重解决的问题是传统云计算（或中央计算）模式下存在的高延迟、网络不稳定和低带宽的问题。

| ![](http://docs.edgegallery.org/zh_CN/latest/_images/EdgeGallery_Architecture.png) |
|:-:|
| EdgeGallery v1.1 版本架构图 |

边缘存储（Edge Storage）就是把数据直接存储在数据采集点，而不需要把采集的数据通过网络（即时）传输到存储的数据中心服务器（或云存储）的数据存储方式。例如：
* 公共监控摄像：在摄像头本地保存数据，即时处理（[Axis](https://www.axis.com/en-us/technologies/edge-storage#:~:text=Edge%20storage%20is%20a%20technology%20used%20in%20Axis,the%20cost%20and%20effort%20for%20remote%20site%20recording.)）
* 家庭数据中心：用户希望数据存储在自己家中，而不是存储与安全防护的公司，希望公司不接触数据，兼顾隐私与安全
* 车联网采集数据：往往可以在端进行先期处理，把整理后的少量数据传给数据中心

| ![星际云通](https://www.szxjyt.cn/static/images/bianbanner2.png) |
|:-:|
| 星际云通总体架构 |

边缘计算适用于与物联网设备紧密相关、数据传输延迟敏感、数据交互次数多、数据传输量大的物联网应用；而云存储则适用于延迟敏感度稍低的互联网应用。

> 一般“边缘计算”指“计算”下沉至“端”场景

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

> 基础设施也必须支持容器化部署。容器运行环境有其本带的问题：
> * 若使用内核驱动，则需要其支持命名空间
>   * 多数硬件接口，包括 iSCSI，暂无支持方案
>   * IOMMU / SR-IOV
> * 天生对系统服务不友好
>   * `/var` 目录管理困难
>   * 内核空间相互隔离
> * 无第一方持久化存储支持
>   * 映射挂载目录 / 设备文件 / 文件镜像

![](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/5GEdge.svg)

| ![](https://object-storage-ca-ymq-1.vexxhost.net/swift/v1/6e4619c416ff4bd19e1c087f27a43eea/www-assets-prod/edge/whitepaper/centraldatacenter.jpg) |
|:-:|
| A detailed view of the edge data center with an automated system used to operate a shrimp farm |

#### 边缘计算模型

> 边缘计算模型一般分为 _控制平面_ 与 _数据平面_ 两个解耦的部分讨论

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

> 云-边半对等模型，根据服务质量需要来相互“卸载”计算任务

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

> 商业模式：用户信息存储不再与某一个服务提供商绑定

### In our context

#### Intuition

网络分块（Partition）更加明显的数据中心。

分块内的网络与分块之间的网络有明显的性能异构性，甚至可能在接入方式上也异构（但该差异通过 NFV
技术消除，似乎与电信行业密切相关）。

在项目的上下文中，_边缘存储_ 更多是指 _边缘数据中心_ 上的分布式持久化存储。

#### 边缘数据中心

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
    >
    > | ![](https://www.h3c.com/cn/res/202012/09/20201209_5411526_image002_1362371_473305_0.jpg) |
    > |:-:|
    > | 新华三云边协同处理流程 |


Network Functions Virutalization (NFV)
-------------------------------------

* [Network Function Virtualization – The Opportunity for OpenStack and Open Source](https://superuser.openstack.org/articles/network-function-virtualization-the-opportunity-for-openstack-and-open-source/)
* [TelcoWorkingGroup - OpenStack](https://wiki.openstack.org/wiki/TelcoWorkingGroup)
* [What is Network Functions Virtualization?](https://thenewstack.io/de-ossify-the-network-with-function-virtualization/)

> 主要与电信（Telco）行业相关

### Background

The networking industry is changing:

* Disaggregation of control plane and data plane

    Taking the current architecture of proprietary, expensive, complex,
    difficult-to-manage forwarding devices (e.g. routers) and SDN aims to "put
    an API on it", forwarding devices become devices controlled by open standards.

* A shift in the telco data-center world which embraces lessons from elastic
    infrastructure cloud

* Disaggregation of hardware and software

    The software part can be open-source implementations of optimized
    packet-forwarding capabilities which used to be implemented in expensive
    and proprietary hardware appliances (or "middleboxes").

The main consumers of NFV are Service providers who are looking to accelerate
the deployment of new network services.

NFV is a complementary initiative to SDN, and SDN makes using NFV much easier and
better.

> 电信提供商希望通过引入云计算中的、基于虚拟化技术的服务管理模式来在基础设施共用的环境下降低运营成本、加速功能迭代
>
> 通过 NFV 技术使用虚拟化消除接入方式的不同

<!--
### NFV in Action

__Use case__

* Virtual customer premise equipment (vCPE)
* Packet switching mobile network
* etc
-->

### NFV Architecture

| ![](https://thenewstack.io/wp-content/uploads/2015/10/NFV-Architecture.jpg) |
|:-:|
| The Architectural Framework proposed by ETSI NFV ISG. The grey lines show the main reference points, where green lines and blue lines show the execution and other reference points, respectively. |

Main blocks of NFV framework are:
* Infrastructure consisting of hardware resources and corresponding virtualization
* Management and Orchestration (MANO), consisting of orchestrator, virtualized
    network functions manager (VNFM) and Virtualized Infrastructure Manager (VIM)
* Virtualized network function and corresponding element management systems (EMS)
* Operating Support System and Business Support Systems (OSS/BSS)

A Virtualized Network Function (VNF) is a Network Function capable of running
on an NFV Infrastructure (NFVI) and being orchestrated by a NFV Orchestrator
(NFVO) and VNF Manager.

The NFVI is the totality of the hardware and software components which build up
the environment in which VNFs are deployed.

NFV decouples software implementations of Network Functions from the compute,
storage, and networking resources through a virtualized layer.

The combination of NFVO, VIM and VNFM is typically referred as MANO. NVFO is
responsible for initialization and setup of new network services, network service
lifecycle management, global resource management, validation and authorization of
requests for NFVI, as well as policy management for network service instances.

VNFMs are responsible for lifecycle management of VNF instances and the overall
coordination between NFVI and EMSs.

VIM, such as OpenStack, is responsible for controlling, managing and monitoring
NFVI compute, storage and network resources.

| ![](https://wiki.opnfv.org/download/attachments/2925118/opnfv_diagram_fraser_042318.png?version=1&modificationDate=1541999962000&api=v2) |
|:-:|
| OPNFV Release Architecture |

> NFV 的具体实现形式尚未知

### 其他材料

* [面向云网融合的细粒度多接入边缘计算架构](https://crad.ict.ac.cn/CN/article/downloadArticleFile.do?attachType=PDF&id=4443)
* [支持网络切片和绿色通信的软件定义虚拟化接入网](https://crad.ict.ac.cn/CN/article/downloadArticleFile.do?attachType=PDF&id=4444)

### 网络拓扑发现 Network Topology Discovery

* [Current Trends of Topology Discovery in OpenFlow-based Software Defined Network](https://academic.microsoft.com/paper/1915875214/citedby/search?q=Current%20Trends%20of%20Topology%20Discovery%20in%20OpenFlow-based%20Software%20Defined%20Networks&qe=RId%253D1915875214&f=&orderBy=0)
    * [Discovering the Network Topology: An Efficient Approach for SDN](https://upcommons.upc.edu/bitstream/handle/2117/103436/Efficient_Approach_SDN.pdf?sequence=6&isAllowed=y)  
        通过 Dijkstra-like 算法找到由网络延迟定义的最短生成路径
* [Efficient topology discovery in software defined networks](https://academic.microsoft.com/paper/2087899556/reference/search?q=Efficient%20topology%20discovery%20in%20software%20defined%20networks&qe=Or(Id%253D2147118406%252CId%253D2022758041%252CId%253D2798915702%252CId%253D2136451165%252CId%253D2186961980%252CId%253D2028926203%252CId%253D1769222792%252CId%253D2110722699%252CId%253D2089939717%252CId%253D2042876290%252CId%253D2222758232)&f=&orderBy=0)

在 SDN 中（基于 OpenFlow 交换机抽象）通常通过二层协议 LLDP（Link Layer Discovery Protocal）实现，SDN 交换机默认支持 LLDP，但网络拓扑发现的计算仍由 SDN 控制器完成，且该实现并未标准化（但通常认为 NOX 的实现，OFDP - OpenFlow Discovery Protocal，为标准实现）。

> __NOTE__
>
> SDN 中不存在“路由器”概念，路由功能（即报文受控转发）由 SDN 交换机完成。

| ![](https://i.loli.net/2021/06/28/1sGa7OimwHFNQcp.png) |
|:-:|
| OFDP 工作流程：控制器通过 Packet-In 报文发现连接 (S1.P1, S2.P3) |

若网络中的传统交换机不支持 LLDP，其二层报文会被直接丢弃。可以利用广播机制来使报文“穿过”传统交换机（如 BDDP，Broadcast Domain Discovery Protocal，非标准协议）。

暂无发现传统交换机的方法，因为传统交换机在逻辑上是透明的。

-------------------------------------------

Implementation and Deployment
=============================

Ceph at the Edge
----------------

### [Use Mars 400 Ceph Storage in Edge Datacenter](https://www.ambedded.com.tw/en/use-case/Use-Mars-400-Ceph-Storage-in-Edge-Datacenter/use-case-01.html)

> * 数据服务器小型化、可热插拔
> * 主要针对私有云场景

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

> OpenStack 边缘计算/超融合架构路线书
>
> 与 Canonical 合作，基于 LXD 容器；官方 Docker 容器支持情况位置

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

* [MaaS](https://maas.io/) to manage bare metal hardware
* [LXD](https://linuxcontainers.org/) clustering to provide an abstract layer of
    virtualization
* Ceph for distributed storage
* MicroK8s to provide a Kubernetes cluster

<!--
### [OpenShift Container Storage: Why Ceph?](https://www.openshift.com/blog/openshift-container-storage-why-ceph)

> What makes Ceph unique - ... - is that it's safe.
-->


TODO:

Ceph with Partitioned Network
-----------------------------

TODO:

### Idea

* CRUSH & 带权图
    * 延迟定义的带权图
    * 模糊指定副本放置位置
        * 确保副本在地理上邻近
        * 确保降级副本离主副本不远
    * [ ] 带权图如何与 CRUSH Map 一起在集群中同步？
        * 必要性：CRUSH 为启发式数据放置算法，必须保证算法的输入在所有节点上一致！
        * 元数据同步开销可能过大，从树变成图
        * 使用 Ceph 的增量 gossip？
    * ...

### Oberservation

* 当 pool 跨多个分区时，分区间延迟对读写性能影响大小依次为 随机读 >> 顺序读 > 随机写。

    * | Inter-partition Lat. | Write IOPS / Lat.  | Seq IOPS / Lat.   | Rand IOPS / Lat. |
      |----------------------|--------------------|-------------------|------------------|
      | +0ms                 | 33 / 0.47          | 377 / 0.04        | 7305 / 0.002     |
      | +50ms                | 24 / 0.65          | 118 / 0.13        | 254 / 0.06       |
        * 延迟加在网络界面上，即 RT 延迟为所加延迟的两倍
        * CRUSH 规则选择 host 作为叶子节点，即三个副本分布在三个节点（也即网络分区）上
        * `rados -p test -b 4K [-O 4M] bench 180 write [-t 16] --no-cleanup`

### Approach

#### Target data placement

对于一个 3-副本的存储池，我们希望其中
* 前两个副本在离计算较近的边缘节点（可能为发起计算的节点），从而降低读延迟
* 第三个副本在离计算较远的边缘节点，保障发生关联故障时的数据安全

#### Vanilla CRUSH approach

```text
 -1         0.04500  root default
 -3         0.01500      host kart-1
  0    hdd  0.00499          osd.0             up   1.00000  1.00000
  1    hdd  0.00499          osd.1             up   1.00000  1.00000
  2    hdd  0.00499          osd.2             up   1.00000  1.00000
 -5         0.01500      host kart-2
  3    hdd  0.00499          osd.3             up   1.00000  1.00000
  4    hdd  0.00499          osd.4             up   1.00000  1.00000
  5    hdd  0.00499          osd.5             up   1.00000  1.00000
 -7         0.01500      host kart-3
  6    hdd  0.00499          osd.6             up   1.00000  1.00000
  7    hdd  0.00499          osd.7             up   1.00000  1.00000
  8    hdd  0.00499          osd.8             up   1.00000  1.00000
```

对于这样集群结构，我们可以在 CRUSH Map 中人为配置与现实情况相符的分区。

```text
 -9         0.01500  region only-kart-1
 -3         0.01500      host kart-1
  0    hdd  0.00499          osd.0             up   1.00000  1.00000
  1    hdd  0.00499          osd.1             up   1.00000  1.00000
  2    hdd  0.00499          osd.2             up   1.00000  1.00000
-11         0.03000  region except-kart-1
 -5         0.01500      host kart-2
  3    hdd  0.00499          osd.3             up   1.00000  1.00000
  4    hdd  0.00499          osd.4             up   1.00000  1.00000
  5    hdd  0.00499          osd.5             up   1.00000  1.00000
 -7         0.01500      host kart-3
  6    hdd  0.00499          osd.6             up   1.00000  1.00000
  7    hdd  0.00499          osd.7             up   1.00000  1.00000
  8    hdd  0.00499          osd.8             up   1.00000  1.00000
```

然后在人为构造的辅助分区上构建规则。

```text
rule kart-1_centric {
        id 1
        type replicated
        min_size 3
        max_size 10
        step take only-kart-1
        step chooseleaf firstn 2 type host
        step emit
        step take except-kart-1
        step chooseleaf firstn 0 type host
        step emit
}
```

* [ ] 近计算分区故障时 OSD 分布
* 对于 n 个分区会产生 O(n) 个辅助分区
* 首次添加新分区的 Bucket 时需要更新所有现有的辅助分区

可以通过脚本生成 CRUSH Map
* 获取当前 CRUSH Map
    * JSON：`ceph osd crush dump`

        ```typescript
        interface CRUSHMap {
            devices: {
                id: number,
                name: string,
                class: "hdd" | "ssd" | "nvme"
            }[],
            types: {
                type_id: number,
                name: string
            }[],
            buckets: {
                id: number,
                name: string,
                type_id: number,
                type_name: string,
                weight: /*16.16 fixed-point*/number,
                alg: "uniform" | "list" | "tree" | "straw" | "straw2",
                hash: "rjenkins1",
                items: {
                    id: number,
                    weight: /*16.16 fixed-point*/number,
                    pos: number
                }[]
            }[],
            rules: {
                rule_id: number,
                rule_name: string,
                ruleset: number,
                type: number,
                min_size: number,
                max_size: number,
                steps: (
                      { op: "take", item: number, item_name: string }
                    | { op: "choose_firstn" | "choose_indep"
                            | "chooseleaf_firstn" | "chooseleaf_indep",
                        num: number, type: string }
                    | { op: "emit" }
                )[]
            }[],
            tunables: any,
            choose_args: any
        }
        ```

    * 二进制：`ceph osd getcrushmap -o <bin>`
* 反编译二进制 CRUSH Map：`crushtool -d <bin> -o <txt>`
* 编译二进制 CRUSH Map：`crushtool -c <txt> -o <bin>`
