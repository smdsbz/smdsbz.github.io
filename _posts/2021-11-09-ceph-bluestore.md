---
layout: article
title: Ceph BlueStore Deep Dive
key: ceph-bluestore-deep-dive
tags: Storage DistributedStorage Ceph BlueStore SourceCode
---

<!-- more -->

Overview
--------

| ![bluestore-metadata](http://sysnote.github.io/2016/08/19/ceph-bluestore/metadata.png) |
|:-:|
| BlueStore metadata |

| ![bluestore-transaction](https://pic2.zhimg.com/80/v2-415968c7317a67864e59dcddf0783815_720w.jpg) |
|:-:|
| BlueStore transaction |

| ![bluestore-state-machine](https://pic2.zhimg.com/80/v2-9d85c11d9c81a4c82128ecacb13020c5_720w.jpg) |
|:-:|
| BlueStore state machine |


Operations
----------

### OP_WRITE

| ![write](http://sysnote.github.io/2016/08/19/ceph-bluestore/writeflow.png) |
|:-:|
| Write procedure |

| ![write-mode](http://sysnote.github.io/2016/08/19/ceph-bluestore/iosplit.png) |
|:-:|
| Write modes |

> `src/os/bluestore/BlueStore.cc/BlueStore::_do_write_data()`


Allocator
---------

### StupidAllocator

`allocate()` repeated calls `allocate_int()`, until allocated size reaches
wanted size.

> `src/os/bluestore/StupidAllocator.cc/StupidAllocator::allocate_int()`

To understand how StupidAllocator works, the data structure
`vector<interval_set<uint64_t, btree_map_t>> free` must be explained:
`free` list, as the name suggests, keeps track of available segments. One thing
special about it is that it's indexed by __segment size__, the B-Tree `free[0]`
manages segments of 1 block size (`bdev_block_size`) long for example.
TODO:


1. `_choose_bin()` - returns a chosen _bin_ `hint` from available segments
    `vector<interval_set<uint64_t, btree_map_t>> free`  
    Given target allocation size `len`, returns the minimum among effective bits
    of `len` and the last element of `free` list.
    * `free` list is a set of B-Trees of available segment, classified by
        magnitude of allocation.
    > The number of entries in `free` is fixed to 10 on initialization, i.e. the
    > maximum contiguous managed allocation block size is `bdev_block_size << 10`.

2. Start searching heuristically in available segments `free` from `hint`




---

* [BlueStore写流程事务实现梳理](https://zhuanlan.zhihu.com/p/387927597)
* [Ceph存储引擎BlueStore解析](http://sysnote.github.io/2016/08/19/ceph-bluestore/)
* [Ceph BlueStore磁盘空间分配初探](https://cloud.tencent.com/developer/article/1492262)
* [BlueStore源码分析之Stupid分配器](https://zhuanlan.zhihu.com/p/91019062)
