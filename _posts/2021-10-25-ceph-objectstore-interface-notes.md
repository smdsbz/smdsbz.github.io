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
  + static create()         // ObjectStore 实例化入口
  + static probe_block_device_fsid()
  |                         // 嗅探该 ObjectStore 实例的 fsid 元数据
  |                         // MemStore 虽有 fsid 元数据，但未在该函数中补充嗅探实现
  |
  |
  | /* 获取该 OSD 性能指标（一般在无锁时调用） */
  + get_cur_stats() = 0     // 获取 ObjectStore 通用性能计数器 objectstore_perf_stat_t
  + get_perf_conters() = 0  // 获取该 ObjectStore 实现特定的 PerfCounters
  |
  |
  + class CollectionImpl : public RefCountedObject  // 对象集合实现虚基类
  |   |
  |   + flush() = 0         // 阻塞直至该集合上的所有事务均提交且可见
  |   + flush_commit() = 0  // [async] 当对象集合所有前置事务完成后执行回调
  |   |                     // 若集合目前没有正在执行的事务，则直接返回 true，传入的回调
  |   |                     // 不会被执行；否则将传入的回调添加到事务完成提交回调队列里面，
  |   |                     // 并返回 false
  |   |                     //
  |   |                     // 源码实现中仅向其传入 C_SaferCond，即仅有通知唤醒线程功能的
  |   |                     // 回调，用于在集合数据状态达成最终一致前休眠本 OSD 线程。
  |   + get_cid()
  |   |
  |   + ...                 // 可在实现中自行补充补充其他对象集合层的 helper（如 finder,
  |                         // reader, writer）
  + using CollectionHandle = ceph::ref_t<CollectionImpl>
  |                         // boost::intrusive_ptr<RefCountedObject>
  |
  |
  + class Transaction       // 见 src/os/Transaction.h
  |   |
  |   + struct Op           // 实际数据结构为封装的 ceph::buffer::list op_bl
  |   + struct TransactionData data
  |   |
  |   + get_object_index()  // [FileStore] 获取对象名-ID 映射
  |   + register_on_{applied|commit|applied_sync|complete}()
  |   |                     // 携带回调函数，之后会由 queue_transactions() 传递给
  |   |                     // ObjectStore Finisher 执行
  |   |
  |   + has_contexts()      // 是否携带了回调函数
  |   + collect_contexts(), get_on_{applied|commit|applied_sync}()
  |   |                     // 抽取回调列表
  |   |
  |   + {set|get}_fadvice_flags(), set_fadvice_flag()
  |   |                     // 当前事务的 I/O 建议 fadvice
  |   |
  |   + append()            // 将另一个事务中的操作拼接到该事务之后
  |   |
  |   + get_encoded_bytes[_test](), get_num_bytes()
  |   + get_data_{length|offset|alignment}()
  |   |                     // 打包后的事务数据结构
  |   + empty()
  |   + get_num_ops()
  |   |
  |   + class iterator      // 打包事务读取器
  |   |   + have_op()
  |   |   + decode_op()     // 将事务的操作列表表头元素取出
  |   |   |
  |   |   + decode_{string|bp|bl|attrset[_bl]|keyset[_bl]}()
  |   |   + get_{oid|cid|fadvice_flags}()
  |   |   |                 // 具体每个 Op 携带的参数，以及这些参数对应封装段见源码
  |   |   + get_objects()
  |   |
  |   + _get_next_op()      // [internal] 一些实现填充操作函数簇的助手函数
  |   + _get_coll_id()
  |   + _get_object_id()
  |   |
  |   | /* 向事务内操作列表中添加实际操作项内容的函数簇，构建
  |   |    ObjectStore::Transaction::Op 并打包填充到本事务中 */
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
  |   + collection_move()   // [deprecated] 与 rename 同义
  |   + collection_move_rename(), try_rename()  // 对象重命名 / 移动
  |   |                     // try_rename() 即同一集合内的 collection_move_rename()
  |   |                     // BlueStore 不支持跨集合 collection_move_rename()
  |   + omap_clear()
  |   + omap_setkeys()
  |   + omap_rmkeys(), omap_rmkeyrange()
  |   + omap_setheader()
  |   + split_collection()  // 将一个集合中满足条件的对象移动到指定的新集合中
  |   |                     // 条件为：对象键哈希的低 bits 位与 rem 相同
  |   |                     // （仅在单元测试中出现，并无实际使用）
  |   + merge_collection()  // 将前一个集合的对象全部迁移至后一个集合中，并设置合并后集合的
  |   |                     // 分裂位 split_bits
  |   + collection_set_bits()
  |   + set_alloc_hint()    // 对某个对象设置分配器偏好
  |   |                     // 如 BlueStore 中通过 flag 来决定是否压缩对象
  |   |
  |   + encode() / decode()
  |   + dump()
  |   + generate_test_instances()   // [debug]
  |
  | /* （异步）（写）操作（事务）执行入口：依次（排队）执行事务，并执行同步回调、注册回调 */
  + queue_transaction[s]()  // 将事务加入执行队列（或直接同步完成开销较小操作，如 MemStore）
  |
  |
  | /* OSD 监控与管理 */
  + upgrade()
  + get_db_statistics()     // 获取 KV 数据库（BlueStore RocksDB）统计数据
  + generate_db_histogram()
  + flush_cache()           // 清空 ObjectStore 层内部缓存
  + dump_perf_counters()
  + dump_cache_stats()
  |
  |
  | /* OSD 运行时管理 */
  + get_type() = 0          // ObjectStore 类型名称
  |
  + test_mount_in_use() = 0 // 当前 ObjectStore 是否正在使用中，即是否被 OSD 挂载
  |                         // BlueStore 通过尝试给 fsid 元数据文件上锁实现
  + mount() = 0             // 挂载 ObjectStore，设置运行时（如启动回调执行线程）
  + umount() = 0            // 卸载 ObjectStore，注销运行时（如等待并结束回调执行线程）
  |
  + fsck()                  // 检查 ObjectStore 内部存储错误
  + repair()                // 修复 **
  + quick_fix()             // 简单修复 **
  |
  + set_cache_shards()      // 设置 ObjectStore 层内部缓存大小
  |
  + validate_hobject_key() = 0      // 检查对象名是否过长
  + get_max_attr_name_length() = 0  // 获取最长 xattr 键长
  |
  + mkfs() = 0              // 格式化，对 ObjectStore 的元数据（如 fsid、type、
  |                         // collections 已有数据）进行初始化或加载，这些元数据通常
  |                         // 以文件的形式存放在设置项 osd_data 指定的目录下
  |
  + mkjournal() = 0         // [FileStore] 单独创建日志区域
  + needs_journal() = 0     // 需要设立日志
  + wants_journal() = 0     // 推荐设立日志
  + allows_journal() = 0    // 允许设立日志
  |                         // BlueStore 不能独立于数据创建日志区（BlueFS, block.wal），
  |                         // 因此全部为 false
  |
  |
  | /* 性能分类 */
  + get_devices()           // 获取当前 ObjectStore 所使用的设备，ObjectStore 未启动
  |                         // 时也应能返回将会使用的设备
  + is_sync_onreadable()    // 事务是否在注册（queue_transactions()）后即可读
  |                         // BlueStore 因为是异地写，在 _txc_add_transaction() 阶段
  |                         // 即为所有写操作创建新 Onode 添加到 onode_map 中，而所有对象
  |                         // 读都需要先获取对象所在 Onode，直接定位到已提交事务
  |                         //
  |                         // 但似乎并不要求是持久化的（f[data]sync()-ed），实现中也并
  |                         // 没有用到
  + is_rotational()
  + is_journal_rotational() // BlueStore 返回 BlueFS 设备类型，尽管 allows_journal() 返回 false
  + get_default_device_class()  // 获取默认设备类型，返回 CRUSHMap 中申明的字符串，如 "pmem"
  |
  + get_numa_node()         // 获取 NUMA 亲和性偏好，由 OSD 层设置
  |                         // BlueStore 支持由 blkdev 层提供
  + can_sort_nibblewise()   // “可半字节排序”，不知道啥意思，实现也没用到。。
  |
  |
  | /* OSD 元数据管理 */
  + statfs() = 0            // 初始化 OSD 层提供的统计数据结构（见 store_statfs_t）
  |                         // 与自定义错误类型
  + pool_statfs() = 0       // 在 OSD 端对每个 pool 统计 statfs，返回 -ENOTSUP 代表不支持
  + collect_metadata()      // ObjectStore 元数据，通常以文件形式存放于 osd_path 下
  + write_meta()
  + read_meta()
  |
  |
  | /* 对象集合句柄相关 */
  + get_ideal_list_max()    // 一个 ObjectStore 最多管理的对象集合的推荐数量
  + open_collection() = 0   // 查询并获取一个对象集合句柄（CollectionHandle）
  |                         // 或创建一个孤立句柄，但一般不会这样，仅出于兼容遗留实现原因提供
  |                         //
  |                         // 对象集合可能是纯内存的（比如仅存在于运行时的超级块、元数据对象）
  + create_new_collection() = 0
  |                         // 为即将被创建的集合分配句柄，之后由事务中 OP_MKCOLL 完成实际创建操作
  |                         //
  |                         // 此外也可以预先构建一些集合运行时，如 BlueStore 在预创建
  |                         // 阶段即绑定 OpSeqencer 事务协调器
  |                         //
  |                         // 上层逻辑会创建临时集合以辅助实现事务（如回滚实现，
  |                         // src/osd/ReplicatedBackend.cc/submit_push_data()，
  |                         // 先在临时集合中异地写，随后将修改的对象覆盖到原集合中，若事务
  |                         // 执行失败，则直接删去临时集合即可，原集合中数据不会丢失，同时
  |                         // 避免脏读），因此可以考虑缓存一些临时集合，不必每次都创建
  + set_collection_commit_queue() = 0
  |                         // 设置某个对象集合完成提交事件的回调队列
  |                         //
  |                         // queue_transactions() 需要将事务携带的对应回调添加到 OSD
  |                         // 负责执行的完成提交回调队列中
  |                         //
  |                         // MemStore 直接在 ObjectStore 的 finisher 中执行回调，
  |                         // 将本函数直接留空。这是因为其数据操作在 queue_transactions()
  |                         // 中就已完成（applied & committed），不会导致不一致
  |
  |
  | /* 同步读操作 */
  + exists() = 0            // 判断对象是否在集合中
  + set_collection_opts() = 0   // 向下传递（初始化的或更新的）pool 设置（见 pool_opts_t），
  |                             // 返回 -EOPNOTSUPP 代表不支持
  + stat() = 0              // 获取对象状态（使用 Linux 原生文件 stat）
  + read() = 0              // 读集合中对象内容
  + fiemap() = 0            // 获取对象数据段布局 file extent mapping (RLE)
  + readv()                 // 批量读，默认实现调用 read()
  + dump_onode()            // [debug] 打印对象信息
  + getattr[s]() = 0        // 获取对象元数据 xattr
  |
  |
  | /* 对象集合操作 */
  + list_collections() = 0  // 列出当前 ObjectStore 上所有对象集合
  + collection_exists() = 0 // 对象集合是否存在
  + collection_empty() = 0  // 对象集合是否为空
  + collection_bits() = 0   // 返回 coll_t::pgid 有效位数（低位表示 PG 中对象过多被切分成多个集合）
  + collection_list[_legacy]() = 0  // 列出对象集合中所有对象
  |
  |
  | /* 对象 Omap 相关 */
  + omap_get() = 0          // 获取对象 KV 数据库
  + omap_get_header() = 0
  + omap_get_keys() = 0
  + omap_get_values() = 0
  + omap_check_keys() = 0   // 检查输入的键有哪些是存在于 omap 上的
  + get_omap_iterator() = 0
  |
  |
  | /* Misc */
  + flush_journal()         // 目前仅 FileStore 实现了独立的日志功能
  + dump_journal()
  + snapshot()              // 目前仅 FileStore 实现了快照功能（依赖底层文件系统实现）
  |
  + {set|get}_fsid() = 0
  |
  + estimate_objects_overhead() = 0
  |                         // 估计存放对象的额外空间消耗
  |                         //
  |                         // PrimaryLogPG 会将该估计值计入已用空间（num_user_bytes），
  |                         // 因此 `ceph df` 的空间结果并不准确（Cache Tier 的提升限流
  |                         // 也是根据平均对象大小进行估计），只有对象数量是准确的
  |
  | /* 调试接口 */
  + inject_data_error()
  + inject_mdata_error()
  |
  + compact()               // 压缩使用空间
  |                         // BlueStore 压缩 RocksDB
  + has_builtin_csum()      // 是否有内建简易数据完整性验证
                            // 若无，则 PrimaryLogPG 会为该对象计算一个 CRC
```

## Key Concepts

### 对象内容及其语义

`ObjectStore` 中存储的对象为命名集合 `coll_t` 中的 `ghobject_t` 或 `hobject_t`。对象支持创建、修改以及删除，集合支持按键顺序遍历。

* 对象名全局唯一，对象数据内容大小限制通常为 100MB 左右。对象数据的稀疏存放不是必须的，但推荐实现。
    * 根据对象支持特性或所处层次不同，对象名（key / oid）会用以下几种表示
        1. `src/include/object.h/object_t` - 对 `std::string name` 的简单封装
        2. `src/include/hobject.h/hobject` - hashed pooled snapped object
        3. `src/include/hobject.h/ghobject` - generationed sharded hobject
* `xattrs` 为对象扩展属性（extended attributes），其本质为键值对集合，不要求实现值内寻址。一个对象的扩展属性通常不超过 64KB，且 Ceph 认为对象内容与其扩展属性通常在物理上临近
* `omap_header` 为一整块数据。
* `omap` 项与扩展属性 `xattr` 类似，但通常存放在不同的地方。`omap` 项的大小可以达到几个 MB，可单独作为一个 KV 数据库工作（CephFS、RGW），因此接口规定必须实现其上的随机寻址。

### 对象集合（Collection）

对象集合（`coll_t`）即为一组对象的集合，与 Placement Group（PG）对应，但当一个 PG 内对象过多时，可能会分裂成多个集合（OP_SPLIT_COLLECTION\[2]，但目前未见源码中有地方调用）。

对象集合也负责对事务进行排序——对同一对象集合进行操作的事务按照顺序依次执行，对不同对象集合进行操作的事务可能并行执行。

> 对同一 PG 操作顺序性同时也由 OSD 层保证：`OSD::op_sharedwq` 在对处理操作时，`OSD::SharedWQ::_process()`
> 对操作涉及 PG 上锁，之后再将操作出队、执行。

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
* [rados objects: omaps and xattrs](https://medium.com/opsops/rados-objects-omaps-and-xattrs-32e66d2b528b)
