---
layout: article
title: Modern Ceph
key: modern-ceph
tags: Storage DistributedStorage PersistentMemory Asynchronous
---

<!-- more -->

* [Ceph Crimson OSD](https://github.com/ceph/ceph-notes/blob/master/crimson/status.rst)
    * Utilize
        * fast networking devices & storage devices
        * multi-core
        * modern techniques
    * Goals
        * bypass kernel
        * avoid memcpy
        * avoid lock contention
* [Ceph SeaStore](https://docs.ceph.com/en/latest/dev/seastore/)


Modern Technologies
-------------------

### SeaStar

A C++ asynchronous programming framework.

* user space task scheduler
    * no context switch to minimize system CPU acquisition

> Ceph's crimson-osd use SeaStar to simplify asynchronous development, and to
> relieve CPU from `if-else`s.


### Data Plane Development Kit (DPDK)

Provide a simple, complete framework for fast packet processing in data plane
applications.

* polling-mode

> Used as PCIe driver (_environment_) in SPDK, for MMIO (Memory-Mapped I/O), PCI
> BAR (Base Address Register), thus enabling NVMe CMB (Controller Memory Buffer)
> and achieving zero-copy, that kind of stuff.


### Storage Performance Development Kit (SPDK)

The bedrock of SPDK is a user space, polled-mode, asynchronous, lockless NVMe
driver.

* user space: no context switch, minimize CPU usage
    * applications issue NVMe commands to device directly
* polled-mode: minimize latency & latency jitter
    * it will fully acquire the assigned CPU
    * interrupt-mode is enabled by hardware, effectively involve kernel

        > IO_URING

    * NVMe (DDIO) ensures polling only checks host memory (cache)
* asynchronous
    * runtimes (_reactors_) are pinned to specific CPU cores

        > If a request were not initially polled by the corresponding CPU /
        > thread of the bounded NVMe queue pair (aka the _owning thread_), it is
        > preferred that the request is forwarded to the correct thread via
        >message passing mechenism, as opposed to introduce locking.

    * coroutines (_`spdk_thread`s_) are scheduled by SPDK or a user-specified
        runtime (_environment_), instead of the operating system

        > The default `static` scheduler just round-robin around reactors (on
        > their designated core), polling for events.

* lockless: ring buffer with CaS

#### Blobstore

For "blob" (or object) storage.

* `include/spdk/blob.h`
* `lib/blob/`

__Device Abstraction__

| Abstraction (low-high) | Size                        | Function              |
|------------------------|-----------------------------|-----------------------|
| Logical block          | 256B / 4KiB                 | Device physical block |
| Page (aka Extent)      | 4KiB                        | Device atomic op      |
| Cluster                | Configurable (default 1MiB) | Object size           |
| Blob                   | Multitude of clusters       | Logical object        |
| Blobstore              |                             |                       |

| Op                        | Atomicity             |
|---------------------------|-----------------------|
| Data writes               | Guaranteed            |
| Blob metadata update      | Manual\* / On-offload |
| Blobstore metadata update | On-offload\**         |

> \* `spdk_blob_sync_md()`
>
> \** If not shutdown properly, it will take some time for the Blobstore to fully
> boot up, but consistency is still guaranteed.

__Metadata__

Stored in memory during runtime.

To avoid locking, a separate (SPDK-)thread is used to handle requests on metadata.
However, it's the caller's responsibility not to mix up metadata requests with
each other and with regular I/O requests.

#### Block Device Layer

A single generic library `lib/bdev`, plus a number of optional modules that
implements various types of block devices (equivalent to device driver in a
traditional operating system).

* Bdev
    * `include/spdk/bdev.h`
    * `lib/bdev/`
* Bdev module
    * `include/spdk/bdev_module.h`
    * `module/bdev/`

Bdevs can be layered! Bdevs that route I/O to other bdevs are often referred to
as virtual bdevs (or vbdevs).

> Fun fact, The `pmem` module internally uses `libpmemblk` from PMDK. Who said
> anything about user space, lockless and no context switching?
> ![pmemblk_write](https://i.loli.net/2021/08/18/nr6EJBbAVeLCiQZ.png)

#### Acceleration Framework

##### Intel I/O Acceleration Technology (I/OAT)

> ... I/OAT allows offloading data movement to dedicated hardware within the
> platform, recalim CPU cycles that would otherwise be spent on tasks like
> `memcpy`... Intel I/OAT can take advantage of PCI-Express
> nontransparent-bridging, which allows movement of memory blocks between two
> different PCIe connected motherboards, thus effectively allowing the movement
> of data between two different computers at nearly the same speed as moving
> data in memory of a single computer...
>
> _from [Fast memcpy with SPDK and Intel I/OAT DMA Engine](https://software.intel.com/content/www/us/en/develop/articles/fast-memcpy-using-spdk-and-ioat-dma-engine.html)_

##### Intel Data Streaming Accelerator (DSA)


### Persistent Memory Development Kit (PMDK)

A collection of libraries for common use cases of SCM.

* libpmem(2): low-level
* libpmemobj: transactional object store
    * libpmemobj++: STL-like programming model
* libpmemkv: key in DRAM, value in PMEM
* libpmemblk: atomically updated block / memory file
* libpmemlog: persistent log file
* libpmemcache: use DRAM as LRU cache of PMEM
* libpmemkind: DRAM as fast tier, PMEM as capatity tier


Source
------

_Based on tag `v16.2.5` (@ `0883bdea7337b95e4b611c768c0279868462204a`)._

* `HAVE_BLUESTORE_PMEM` Compile Flag
    * pmem used as `mmap()`-ed file
* `Allocator` type
    * `BlueStore::shared_alloc` defaults to `"block"` (see `src/os/bluestore/BlueStore.cc/BlueStore::_create_alloc()`)

> For all Ceph config options, see `src/common/options.cc/get_global_options()`.


Readings
--------

* [[SDC'16] Challenges in Using Persistent Memory in Distributed Storage Systems](https://www.snia.org/sites/default/files/SDC/2016/presentations/persistent_memory/Dan_Lambright_Challenges_Persistent_Memory_Distributed_Storage_Systems.pdf)
    * High performance network (RDMA) and fast storage devices (SCM) only helps
        with large I/Os, not small
        * LOOKUP RTT dominates faster data transfer
    * Considering streamline, coalescing protocal, shrinking stack etc.

<!--
* [[ATC'17] Octopus: an RDMA-enabled Distributed Persistent Memory File System](https://www.usenix.org/system/files/conference/atc17/atc17-lu.pdf)
    * Leverage RDMA `write_with_imm` and `cmp_and_swp` one-sided verbs to reduce
        waiting on network operation acknowledgements
    * Use shared remote memory pool as view of the overall storage namespace
    * Transactions are done locally, and then pushed to involved nodes, to avoid
        complex communications and coordinations
    * not much about persistent memory...
-->
