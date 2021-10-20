---
layout: article
title: Ceph Cache Tiering and Objecter
key: ceph-cache-tiering-and-objecter
tags: Ceph CacheTiering Objecter Librados RADOS
---

<!-- more -->

| ![librados-procedure](https://img-blog.csdn.net/20171204102701858?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ1NORF9QQU4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) |
|:-:|
| librados request procedure |

Objecter is located at Osdc.

Before sending a request to remote OSDService, it first need to determine which
OSD, or rather PrimaryLogPG, it should talk to. Since the client side issue
requests via the `Ioctx` context, which contains `pg_pool_t`, it already knows
the `read/write_tier` of the base pool on which the `Ioctx` is initialized. So
it does just that!  
__If issuing request to a pool configured with cache tier, librados always talks
to the tier pool, only the PrimaryLogPG on the tier pool talks to the base pool.__

> The corresponding source code can be found at `src/osdc/Objecter.cc::_calc_target()`.

---

* [Ceph学习——Librados与Osdc实现源码解析](https://blog.csdn.net/CSND_PAN/article/details/78707756)
* [ceph的cache tier实现分析](https://blog.csdn.net/u010487568/article/details/91966082)
