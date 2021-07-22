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
