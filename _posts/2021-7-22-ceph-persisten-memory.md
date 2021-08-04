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

### Data Plane Development Kit (DPDK)

Provide a simple, complete framework for fast packet processing in data plane
applications.

### Storage Performance Development Kit (SPDK)

The bedrock of SPDK is a user space, polled-mode, asynchronous, lockless NVMe
driver.

* user space & polled-mode: no context switch
* lockless: ring buffer with CaS

### Persistent Memory Development Kit (PMDK)

A collection of libraries for operating NVDIMM.
