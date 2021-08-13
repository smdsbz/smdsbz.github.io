---
layout: article
title: Ceph & Persistent Memory
key: ceph-persistent-memory
tags: Storage DistributedStorage PersistentMemory
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
* lockless: ring buffer with CaS

#### Blobstore

For "blob" (or object) storage.

* `src/spdk/include/blob.h`
* `src/spdk/lib/blob/`

__Device Abstraction__

| Abstraction (low-high) | Size                        | Function              |
|------------------------|-----------------------------|-----------------------|
| Logical block          | 256B / 4KiB                 | Device physical block |
| Page                   | 4KiB                        | Device atomic op      |
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

__Channel__

SPDK-wide abstraction responsible for actual I/O operations. Best deployed per
thread.

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
