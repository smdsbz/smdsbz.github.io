---
layout: article
title: Ceph CRUSH Pipeline
key: ceph-crush-pipeline
tags: DistributedSystems Storage Ceph CRUSH
---

<!-- more -->

_based on version nautilus (b0c68711039276c1e8d5bfa838207468a36a165c)_


## Intuition

![](https://blog-10039692.file.myqcloud.com/1510039997413_2100_1510040043286.jpg)

> Image explained pipeline for default RBD files, no pool related details are shown.


## Dive-In

1.  `src/osd/OSDMap.h/OSDMap/map_to_pg()`

    ```c++
      int map_to_pg(
        int64_t pool,
        const string& name,
        const string& key,
        const string& nspace,
        pg_t *pg) const;
    ```

    `src/osd/OSDMap.h/OSDMap/object_locator_to_pg()`

    Helper function of `OSDMap::map_to_pg()`, used more often.

    ```c++
      int object_locator_to_pg(const object_t& oid, const object_locator_t& loc,
    			   pg_t &pg) const;
    ```

2. `src/osd/OSDMap.h/OSDMap/raw_pg_to_pg()`

    ```c++
      pg_t raw_pg_to_pg(pg_t pg) const {
        auto p = pools.find(pg.pool());
        ceph_assert(p != pools.end());
        return p->second.raw_pg_to_pg(pg);
      }
    ```

3. `src/osd/osd_types.h/pg_pool_t/raw_pg_to_pg()`

    ```c++
      /*
       * map a raw pg (with full precision ps) into an actual pg, for storage
       */
      pg_t raw_pg_to_pg(pg_t pg) const;
    ```

4. `src/osd/OSDMap.h/OSDMap/pg_to_up_acting_osds()`

    ```c++
      /**
       * map a pg to its acting set as well as its up set. You must use
       * the acting set for data mapping purposes, but some users will
       * also find the up set useful for things like deciding what to
       * set as pg_temp.
       * Each of these pointers must be non-NULL.
       */
      void pg_to_up_acting_osds(pg_t pg, vector<int> *up, int *up_primary,
                                vector<int> *acting, int *acting_primary) const {
        _pg_to_up_acting_osds(pg, up, up_primary, acting, acting_primary);
      }
    ```
