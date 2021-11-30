---
layout: article
title: Persistent Memory Allocator
key: persistent-memory-allocator
tags: PersistentMemory MemoryAllocator SourceCode
---

<!-- more -->

PMDK
----

* [Persistent allocator design - fragmentation](https://pmem.io/2016/02/25/fragmentation.html)

`src/libpmemobj/palloc.c`

_also see Doug Lea malloc, jemalloc and tcmalloc._

__Observation__

Performance dominated by cacheline flush to persistent memory,
i.e. `pmem_persist()`, bit more operations on volatile data won't hurt.

Therefore adopts a hybrid approach, keep track of memory blocks in both transient
(volative) and persistent memory.

__Allocation__

* For large memory blocks (256KiB) best-fit algorithm is used, coalesing not
    deffered.
* For smaller sizes a segregated-fit algorithm is employed, 35 allocations classes.
    The first time a memory block from a class needed, an entire chunk (256KiB)
    is split into smaller blocks of 8x class's size.

    > E.g. first allocation of 128B class, a 256KiB chunk is taken, this will
    > give you 256 available blocks, with each block of size 8*128B=1024B. Blocks
    > you don't need this time will be inserted into a per-class tree.

    Block size is chosen to be 8x class size, because it matches bitmap
    representation, which will be the on-media persistent representation.

__Source Code__

1. `pmalloc()`

    Generates an `OPERATION_INTERNAL` (and locks its _lane_), then pass the
    operation to `palloc`.

    > The introduction of operation and redo-log is needed, for PMEM pool
    > metadata manipulations should be consistent.

2. `palloc_operation()`

    _copied from source code_

    Simplified allocation process flow is as follows:
    - reserve a new block in the transient heap
    - prepare the new block
    - create redo log of required modifications
        - chunk metadata
        - offset of the new object
    - commit and process the redo log

    And similarly, the deallocation process:
    - create redo log of required modifications
        - reverse the chunk metadata back to the 'free' state
        - set the destination of the object offset to zero
    - commit and process the redo log

    There's an important distinction in the deallocation process - it does not
    return the memory block to the transient container. That is done once no more
    memory is available.

    __Parameters__
    * `struct palloc_heap *heap` - `struct pmemobjpool::heap` instance
    * `uint64_t off` - used when free / realloc
    * `uint64_t *dest_off` - [out] fresh allocated memory offset, will be updated
        consistently
    * `size_t size` - requested size
    * `palloc_constr constructor` - left NULL
    * `void *arg` - left NULL
    * `uint64_t extra_field`
    * `uint16_t object_flags`
    * `uint16_t class_id` - default 0, let PMDK decide `heap_get_best_class()`,
        adopts an immediate-fit schema
    * `uint16_t arena_id` - default 0, let PMDK decide `heap_thread_arena()`,
        adopts a least-used schema
    * `struct operation_context *ctx` - the memory operation to perform

    __Return Value__
    * 0 - ok
    * -1 - errno is set

    __Steps__
    1. create ops
        * alloc: `palloc_reservation_create()` reserve _bucket_ in transient heap
            1. get alloc class (from `heap->heap_rt` heap runtime)

                > The default alloc class is managed separately as `rt->default_bucket`.

                > alloc class created with `alloc_class.c/alloc_class_collection_new()`
                > during `heap.c/heap_boot()`.

            2. `heap_bucket_acquire()` get alloc class (bucket) on thread (arena)
            3. `heap_get_bestfit_block()` extract block from bucket
                1. repeatedly `get_rm_bestfit()`
                    * ravl: TODO:
                    * seglists: TODO:
                2. `heap_split_block()` if extracted whole block is wasteful
            4. `alloc_prep_block()` reserve bucket in transient state
                1. `write_header()` see `memblock.h`
                    * MEMORY_BLOCK_HUGE, MEMORY_BLOCK_RUN
                        * HEADER_LEGACY
                        * HEADER_COMPACT
                        * HEADER_NONE

        * free: TODO:

    2. `palloc_exec_actions()`
        1. `action_funcs[atype]->exec()`
            * POBJ_ACTION_TYPE_HEAP -> `palloc_heap_action_exec()` -> `act->m.mops->prep_hdr()`
                * MEMORY_BLOCK_HUGE
                * MEMORY_BLOCK_RUN: prepare bitmap
            * POBJ_ACTION_TYPE_MEM
        2. `pmemops_drain()`
        3. `operation_process()`
            * `ulog_entry_apply()`
            * `operation_process_persistent_redo()`
                1. `ulog_store()` persistent `memcpy()` ulog (undo-log) with checksum
                2. `ulog_process()` for each ulog do a callback
                3. `ulog_clobber()` zero-out ulog metadata
            * `operation_process_persistent_undo()`
                1. `ulog_process()`

        4. unlock & `action_funcs[atype]->on_unlock()`
        5. `operation_finish()`






Poseidon [Middleware'20]
------------------------

* Per-CPU sub-heap for scalability and high-performance

* Undo log for singleton alloc, micro log for transactional alloc, truncated on
    success
* Buddy list keeps track of free segments
* Hash table to manage memory block information (as opposed to AVL)
    * [F2FS](https://www.usenix.org/conference/fast15/technical-sessions/presentation/lee)


ArchTM [[FAST'21](https://www.usenix.org/conference/fast21/presentation/wu-kai)]
--------------------------------------------------------------------------------

* PMDK Libpmemobj's transaction incurs to many metadata update, causing poor
    performance
* Small writes (< 256KiB) causes CoW, waste bandwidth and delays transactions


------------------------------------------------------------------------

* [AEP 入门指北](https://csomnia.github.io/posts/aep-tutorial/)
