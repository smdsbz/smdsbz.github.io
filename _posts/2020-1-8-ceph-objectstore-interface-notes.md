---
layout: article
title: Ceph ObjectStore Interface Notes
key: ceph-objectstore-interface-notes
tags: DistributedSystems Ceph Storage SourceCode
---

<!-- more -->

# Ceph `ObjectStore` Interface Notes

## Class Structure

```text
class ObjectStore
  |
  + static create()         // ObjectStore 对象实例化入口
  + probe_block_device_fsid()
  |
  | /* 获取该 OSD 性能指标（无锁） */
  + get_cur_stats() = 0     // 获取性能计数器 objectstore_perf_stat_t
  + get_perf_conters() = 0
  |
  + class CollectionImpl : public RefCountedObject
  |   |
  |   + flush() = 0         // 阻塞直至所有事务均提交
  |   + flush_commit() = 0  // [异步] 当对象集合所有前置事务完成后执行回调
  |   + get_cid()
  |
  + class Transaction       // 见 src/os/Transaction.h
  |   |
  |   + struct Op
  |   + struct TransactionData
  |   |
  |   + get_object_index()
  |   + register_on_{applied|commit|applied_sync|complete}()
  |   |
  |   + has_contexts()
  |   + collect_contexts()
  |   + get_on_{applied|commit|applied_sync}()
  |   |
  |   + {set|get}_fadvice_flags(), set_fadvice_flag()
  |   |
  |   + _update_op(), _update_op_bl()
  |   |
  |   + append()
  |   |
  |   + get_encoded_bytes[_test]()
  |   + get_num_bytes()
  |   + get_data_{length|offset|alignment}()
  |   |
  |   + empty()
  |   + get_num_ops()
  |   |
  |   + class iterator
  |   + _build_actions_from_tbl()
  |   + _get_next_op()
  |   + _get_coll_id()
  |   + _get_object_id()
  |   |
  |   | /* 向事务内操作列表中添加实际操作项内容的函数簇，构建
  |   |    ObjectStore::Transaction::Op */
  |   + nop()
  |   + touch()             // i.e. create
  |   + write()
  |   + zero()
  |   + truncate()
  |   + remove()
  |   + setattr[s]()
  |   + rmattr[s]()
  |   + clone[_range]()
  |   + create_collection()
  |   + collection_hint()   // 目前只有一种 COLL_HINT_EXPECTED_NUM_OBJECTS
  |   |                     //            (pg_num, expected_num_objects)
  |   + remove_collection()
  |   + collection_move()   // [弃用] 与 rename 同义
  |   + collection_move_rename(), try_rename()  // 对象重命名 / 移动
  |   |                     // try_rename() 即同一集合内的 collection_move_rename()
  |   |                     // BlueStore 不支持跨集合 collection_move_rename()
  |   + omap_clear()
  |   + omap_setkeys()
  |   + omap_rmkeys(), omap_rmkeyrange()
  |   + omap_setheader()
  |   + split_collection()  // 将一个集合中【满足条件的】对象移动到指定的新集合中
  |   +                     // （仅在单元测试中出现，无实际使用）
  |   + merge_collection()
  |   + collection_set_bits()
  |   + set_alloc_hint()
  |   |
  |   + encode() / decode()
  |   + dump()
  |   + generate_test_instances()
  |
  | /* （异步）（写）操作（事务）执行入口：依次（排队）执行事务，并执行同步回调、注册回调 */
  + queue_transaction[s]()  // 将事务加入执行队列（或直接同步完成开销较小操作，如 MemStore）
  |
  | /* OSD 管理 */
  + upgrade()
  + get_db_statistics()
  + generate_db_histogram()
  + flush_cache()
  + dump_perf_counters()
  + dump_cache_stats()
  |
  | /* OSD 运行时管理 */
  + get_type() = 0          // ObjectStore 类型名称
  + test_mount_in_use() = 0
  + mount() = 0             // 挂载 ObjectStore，设置运行时（如启动回调执行线程）
  + umount() = 0            // 卸载 ObjectStore，注销运行时（如等待并结束回调执行线程）
  + fsck()
  + repair()
  + quick_fix()
  + set_cache_shards()
  |
  + validate_hobject_key() = 0
  + get_max_attr_name_length() = 0
  |
  + mkfs() = 0              // 格式化，对 ObjectStore 的元数据（如 fsid、type、
  |                         // collections 已有数据）进行初始化或加载，这些元数据通常
  |                         // 以文件的形式存放在设置项 osd_data 指定的目录下
  + mkjournal() = 0
  + needs_journal() = 0     // 需要设立日志
  + wants_journal() = 0     // 推荐设立日志
  + allows_journal() = 0    // 允许设立日志
  |
  | /* 设备性能分类 */
  + get_devices()
  + is_sync_onreadable()
  + is_rotational()
  + is_journal_rotational()
  + get_default_device_class()
  |
  + get_numa_node()
  + can_sort_nibblewise()
  |
  | /* OSD 元数据管理 */
  + statfs() = 0            // 初始化 OSD 层提供的统计数据结构（见 store_statfs_t）
  |                         // 与自定义错误类型
  + pool_statfs() = 0       // 在 OSD 端对每个 pool 统计 statfs，返回 -ENOTSUP 代表不支持
  + collect_metadata()
  + write_meta()
  + read_meta()
  |
  | /* 对象集合句柄相关 */
  + get_ideal_list_max()
  + open_collection() = 0
  + create_new_collection() = 0
  |                         // 为即将被创建的集合分配句柄，之后由事务中 OP_MKCOLL 完成实际创建操作
  |                         //
  |                         // 上层逻辑会创建临时集合以辅助实现事务（如回滚实现，
  |                         // src/osd/ReplicatedBackend.cc/submit_push_data()，
  |                         // 先在临时集合中异地写，随后将修改的对象覆盖到原集合中，若事务
  |                         // 执行失败，则直接删去临时集合即可，原集合中数据不会丢失，同时
  |                         // 避免脏读），因此可以考虑缓存一些临时集合，不必每次都创建
  + set_collection_commit_queue() = 0
  |
  | /* 同步读操作 */
  + exists() = 0            // 判断对象是否在集合中
  + set_collection_opts() = 0   // 向下传递（初始化的或更新的）pool 设置（见 pool_opts_t），
  |                             // 返回 -EOPNOTSUPP 代表不支持
  + stat() = 0              // 获取对象状态（Linux 原生文件 stat）
  + read() = 0              // 读集合中对象内容
  + fiemap() = 0            // 获取对象数据段布局 file extent mapping (RLE)
  + readv()                 // 批量读，默认实现调用 read()
  + dump_onode()            // [debug] 打印对象信息
  + getattr() = 0, getattrs() = 0   // 获取对象元数据 xattr
  |
  + /* 对象集合 */
  + list_collections() = 0  // 列出当前 ObjectStore 上所有对象集合
  + collection_exists() = 0     // 对象集合是否存在
  + collection_empty() = 0  // 对象集合是否为空
  + collection_bits() = 0   // 返回 coll_t::pgid 有效位数（低位表示 PG 中对象过多被切分成多个集合）
  + collection_list[_legacy]() = 0  // 列出对象集合中所有对象
  |
  | /* 对象 Omap 相关 */
  + omap_get() = 0          // 获取对象 KV 数据库
  + omap_get_header() = 0
  + omap_get_keys() = 0
  + omap_get_values() = 0
  + omap_check_keys() = 0   // 检查输入的键有哪些是存在于 omap 上的
  + get_omap_iterator() = 0
  |
  | /* Misc */
  + flush_journal()
  + dump_journal()
  + snapshot()
  |
  + set_fsid() = 0
  + get_fsid() = 0
  |
  + estimate_objects_overhead() = 0
  |
  | /* 调试接口 */
  + inject_data_error()
  + inject_mdata_error()
  |
  + compact()
  + has_builtin_csum()
```

## Key Concepts

### 对象内容及其语义

`ObjectStore` 中存储的对象为命名集合 `coll_t` 中的 `ghobject_t` 或 `hobject_t`。对象支持创建、修改以及删除，集合支持按键顺序遍历。

* 对象名全局唯一，对象数据内容大小限制通常为 100MB 左右。对象数据的稀疏存放不是必须的，但推荐实现。
    * 根据对象支持特性或所处层次不同，对象名（key / oid）会用以下几种表示
        1. `src/include/object.h/object_t` - 对 `std::string name` 的简单封装
        2. `src/include/hobject.h/hobject` - hashed pooled snapped object
        3. `src/include/hobject.h/ghobject` - generationed sharded hobject
* `xattrs` 为对象扩展属性（extended attributes），其本质为键值对集合，不要求实现值内寻址。一个对象的扩展属性通常不超过 64KB，且 Ceph 认为对象内容与其扩展属性通常在物理上临近。
* `omap_header` 为一整块数据。
* `omap` 项与扩展属性 `xattr` 类似，但通常存放在不同的地方。`omap` 项的大小可以达到几个 MB，因此接口规定必须实现其上的随机寻址。

### 对象集合（Collection）

对象集合（`coll_t`）即为一组对象的集合，与 Placement Group（PG）对应，但当一个 PG 内对象过多时，可能会分裂成多个集合（OP_SPLIT_COLLECTION\[2]，但目前未见源码中有地方调用）。

* 集合支持遍历。
* 集合拥有自己的扩展属性 `xattrs`。

对象集合也负责对事务进行排序——对同一对象集合进行操作的事务按照顺序依次执行，对不同对象集合进行操作的事务可能并行执行。

### 事务

事务（transaction）代表一系列原始数据修改操作。

一个事务在其生命周期中需要执行下列三类回调函数（`Context` 对象）：

1. `on_applied_sync`
2. `on_applied`
3. `on_commit`

`on_applied_sync` 与 `on_applied` 回调函数中的操作允许数据修改对后续 `ObjectStore` 操作可见，即可读的。`on_applied_sync` 与 `on_applied` 的区别仅在于是否加全局锁。`on_applied_sync` 被 `ObjectStore` 主线程直接调用，因此其必须尽快完成执行，且不得依赖执行环境中的任何锁。相对地，`on_applied` 在独立的 `Finisher` 线程中执行，因此可以放心地等待执行环境中的其他锁。

> `on_applied*` 有时也记作 `on_readable*`。

`on_commit` 回调函数在 `Finisher` 线程中执行，其标志着该事务中的所有数据修改已经（高可用地）持久化。

`Transaction` 已经实现了日志逻辑，当然也可以自行另外实现日志（如 `FileStore`）。

> **事务隔离性**
> 
> 除上述隔离性保证，其余性质均由调用者保证（即调用者需要保证在 `on_applied_sync` 之前不读取修改内容）。
>
> 遍历操作的隔离性可能被破坏，其实际行为完全由 `ObjectStore` 实现决定。

-------------------------------------------------------------

* [Ceph OSD request 分析](http://www.yangguanjun.com/2015/06/18/ceph-osd-requst-analysis/)
