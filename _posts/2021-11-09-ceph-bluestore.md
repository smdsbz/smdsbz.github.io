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

To understand how StupidAllocator works, the private member
```c++
std::vector<ceph::interval_set<
    /*offset/length type*/uint64_t,
    /*map impl*/btree_map<uint64_t/*offset*/, uint64_t/*length*/>
    >> free;
```
must be explained:
`free` list, as the name suggests, keeps track of available segments. The vector
is indexed by __magnitude of segment size__, that is `free[0]` will be available
segments of \[0, 1) block size (`bdev_block_size`) and `free[3]` will be of
\[4, 8) bs segments. Since `interval_set` is an
[AssociativeContainer](https://en.cppreference.com/w/cpp/named_req/AssociativeContainer),
the segments in a free list entry is naturally sorted by offset.

> The number of entries in `free` is fixed to 10 on initialization, i.e. the
> maximum contiguous managed allocation block size is `bdev_block_size << 9`.

> `btree_map_t` is like `std::map`, but implemented with B-Tree, rather than
> red-black tree, for smaller footprint.

StupidAllocator then acts as a buddy allocator:

1. `_choose_bin()` - returns a chosen _bin_ `orig_bin` from available segments

    Given target allocation size `len`, returns the minimum among effective bits
    of `len` and the last element of `free` list.

    > PERF: Implemented with `__builtin_clz(ll)` Count Leading Zero instruction.

2. For segments no smaller than `orig_bin`s, i.e. entries after and including
    `free[orig_bin]`, search heuristically from `hint` address.
    > The default hint is the immediate address after last allocation.
3. For segments no smaller than `orig_bin`s, search from lowest address (up to
    `hint`, because already searched).
4. For segments smaller than `orig_bin`s, xxxx
    > Allocate something at least.

| ![](https://pic4.zhimg.com/80/v2-affb31dfc772140d180cdde8bc2f41e7_720w.jpg) |
|:-:|
| Heuristic search range and its order (`bin_start` is `orig_bin`) |

5. Manage `free`.


### HybridAllocator





---

* [BlueStore写流程事务实现梳理](https://zhuanlan.zhihu.com/p/387927597)
* [Ceph存储引擎BlueStore解析](http://sysnote.github.io/2016/08/19/ceph-bluestore/)
* [Ceph BlueStore磁盘空间分配初探](https://cloud.tencent.com/developer/article/1492262)
* [BlueStore源码分析之Stupid分配器](https://zhuanlan.zhihu.com/p/91019062)
