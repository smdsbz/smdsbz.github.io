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

> Image above explained pipeline for default RBD files, no pool related details are shown.


## Dive-In

1.  `src/osd/OSDMap.h/OSDMap::map_to_pg()`

    ```c++
      int map_to_pg(
        int64_t pool,
        const string& name,
        const string& key,
        const string& nspace,
        pg_t *pg) const;
    ```

    `src/osd/OSDMap.h/OSDMap::object_locator_to_pg()`

    Helper function of `OSDMap::map_to_pg()`, used more often.

    ```c++
      int object_locator_to_pg(const object_t& oid, const object_locator_t& loc,
                               pg_t &pg) const;
      pg_t object_locator_to_pg(const object_t& oid,
                                const object_locator_t& loc) const {
        pg_t pg;
        int ret = object_locator_to_pg(oid, loc, pg);
        ceph_assert(ret == 0);
        return pg;
      }
    ```

    `src/osd/osd_types.h/pg_pool_t::hash_key()`

    ```c++
      /// hash a object name+namespace key to a hash position
      uint32_t hash_key(const string& key, const string& ns) const;
    ```

2. `src/osd/OSDMap.h/OSDMap::raw_pg_to_pg()`

    ```c++
      pg_t raw_pg_to_pg(pg_t pg) const {
        auto p = pools.find(pg.pool());
        ceph_assert(p != pools.end());
        return p->second.raw_pg_to_pg(pg);
      }
    ```

   `src/osd/osd_types.h/pg_pool_t::raw_pg_to_pg()`

    ```c++
      /*
       * map a raw pg (with full precision ps) into an actual pg, for storage
       */
      pg_t raw_pg_to_pg(pg_t pg) const;
    ```

3. `src/osd/OSDMap.h/OSDMap::pg_to_up_acting_osds()`

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

    `src/osd/OSDMap.h/OSDMap::_pg_to_up_acting_osds()`

    ```c++
      /**
       *  map to up and acting. Fills in whatever fields are non-NULL.
       */
      void _pg_to_up_acting_osds(const pg_t& pg, vector<int> *up, int *up_primary,
                                 vector<int> *acting, int *acting_primary,
                                 bool raw_pg_to_pg = true) const;
    ```

    `src/osd/OSDMap.h/OSDMap::_get_temp_osds()`

    > `temp` here means _under calculation_ or _to be used for calculation_,
    > this function spits out intermediate results for the calculated physical
    > OSD placements.

    ```c++
      /**
       * Get the pg and primary temp, if they are specified.
       * @param temp_pg [out] Will be empty or contain the temp PG mapping on return
       * @param temp_primary [out] Will be the value in primary_temp, or a value derived
       * from the pg_temp (if specified), or -1 if you should use the calculated (up_)primary.
       */
      void _get_temp_osds(const pg_pool_t& pool, pg_t pg,
                          vector<int> *temp_pg, int *temp_primary) const;
    ```
