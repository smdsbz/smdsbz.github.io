---
layout: article
title: Ceph Objecter bufferlist Format
key: ceph-objecter-bufferlist
tags: Storage DistributedStorage Ceph SourceCode MDS
---

How calling parameters and return data are encoded in Ceph Objecter RPC.

_<small>based on Ceph v16.2.5</small>_

<!-- more -->

Code Teardown Reminder
----------------------

* Encoding procedure of invocation can be found at `src\osdc\Objecter.h\Objecter::prepare_<op>_op()` and `src\osdc\Objecter.h\ObjectOperation`.
* Decoding procedure of returned data can be referenced at `src\osdc\Objecter.h\ObjectOperation::struct CB_ObjectOperation_<op>::operator()()` or `src\osdc\Objecter.h\Objecter::C_<Op>::finish()`, which may very likely be in the same order as they were encoded in `src\osd\PrimaryLogPG.cc\PrimaryLogPG::do_osd_ops()`.

Key Data Structures
-------------------

__Packaged OSD Operations__

```c++
struct OSDOp;
using osdc_opvec = boost::container::small_vector<OSDOp, osdc_opvec_len>;

/* src/osdc/Objecter.h */

struct ObjectOperation {
    osdc_opvec ops;     // OSDOp[]
    int flags = 0;
    int priority = 0;

    boost::container::small_vector<ceph::buffer::list*, osdc_opvec_len> out_bl;
    boost::container::small_vector<
        fu2::unique_function<void(boost::system::error_code, int,
                                    const ceph::buffer::list& bl) &&>,
        osdc_opvec_len> out_handler;
    boost::container::small_vector<int*, osdc_opvec_len> out_rval;
    boost::container::small_vector<boost::system::error_code*,
                                    osdc_opvec_len> out_ec;

    /* ... */
};  /* struct ObjectOperation */

namespace Objecter {

struct op_target_t {
    /* ... */

    /** the key/ID of object to operate on */
    object_t base_oid;              // { std::string name; }
    /** provides the pool in which the object resides */
    object_locator_t base_oloc;     // { int64_t pool; ... }
    /* to be calculated with Objecter::_calc_target() */
    object_t target_oid;
    object_locator_t target_oloc;

    /* ... */
};  /* struct op_target_t */

struct Op {
    /* ... */

    op_target_t target;

    /* ... */

    /** in short OSDOp[], with optimized allocation code path */
    osdc_opvec ops;

    snapid_t snapid = CEPH_NOSNAP;
    SnapContext snapc;
    /** modify time */
    ceph::real_time mtime;

    /**
     * legacy culmulative output
     *
     * Used only by certain read ops (getxattr, read, stat), and only when the
     * overall `Op` has **only one** OSDOp. The content of the only effective
     * `outdata` is then copied to the upper-layer-supplied bufferlist pointed
     * to by this pointer (transfered over the network by the dedicated
     * `Message::data` frame segment).
     */
    ceph::buffer::list *outbl = nullptr;
    /**
     * individual output of each member op, which are just pointers to underlying
     * fields of individual ops. zeroed by default, same size as `ops`
     */
    boost::container::small_vector<ceph::buffer::list*, osdc_opvec_len> out_bl;
    /**
     * immediate callbacks, executed right after receiving reply
     * @sa Objecter::handle_osd_op_reply(MOSDOpReply*)
     */
    boost::container::small_vector<
        fu2::unique_function<void(boost::system::error_code, int,
                                    const ceph::buffer::list& bl) &&>,
        osdc_opvec_len> out_handler;
    boost::container::small_vector<int*, osdc_opvec_len> out_rval;
    boost::container::small_vector<boost::system::error_code*,
                                    osdc_opvec_len> out_ec;

    /* ... */
};  /* struct Op */

}   /* namespace Objecter */
```

__Individual OSD Operation__

```c++
struct ceph_osd_op;
struct sobject_t;

/* src/osd/osd_types.h */

struct OSDOp {
    ceph_osd_op op;     // opcode, flag and extra calling args
    sobject_t soid;     // stripped object name, not used by CephFS, for it has
                        // its own stripping (or they call layout) logic
                        // (see src/osdc/Striper.cc/Striper::file_to_extents()).
                        // and a metadata object always has object size equal to
                        // strip size, which is 4MiB

    /* `outdata` (should be) pointed to by `out_bl`s, if need to claim return
        data, so we may just stuff return data here, should be safe (?) */
    ceph::buffer::list indata, outdata;
    errorcode32_t rval = 0;

    /* ... */
};  /* struct OSDOp */


/* src/include/rados.h */

struct ceph_osd_op {
	__le16 op;           /* CEPH_OSD_OP_* */
	__le32 flags;        /* CEPH_OSD_OP_FLAG_* */
	union {
		struct {
			__le64 offset, length;
			__le64 truncate_size;
			__le32 truncate_seq;
		} __attribute__ ((packed)) extent;
		struct {
			__le32 name_len;
			__le32 value_len;
			__u8 cmp_op;       /* CEPH_OSD_CMPXATTR_OP_* */
			__u8 cmp_mode;     /* CEPH_OSD_CMPXATTR_MODE_* */
		} __attribute__ ((packed)) xattr;
		struct {
			__u8 class_len;
			__u8 method_len;
			__u8 argc;
			__le32 indata_len;
		} __attribute__ ((packed)) cls;
		struct {
			__le64 count;
			__le32 start_epoch; /* for the pgls sequence */
		} __attribute__ ((packed)) pgls;
	        struct {
		        __le64 snapid;
	        } __attribute__ ((packed)) snap;
		struct {
			__le64 cookie;
			__le64 ver;     /* no longer used */
			__u8 op;	/* CEPH_OSD_WATCH_OP_* */
			__u32 gen;      /* registration generation */
			__u32 timeout; /* connection timeout */
		} __attribute__ ((packed)) watch;
		struct {
			__le64 cookie;
		} __attribute__ ((packed)) notify;
		struct {
			__le64 unused;
			__le64 ver;
		} __attribute__ ((packed)) assert_ver;
		struct {
			__le64 offset, length;
			__le64 src_offset;
		} __attribute__ ((packed)) clonerange;
		struct {
			__le64 max;     /* max data in reply */
		} __attribute__ ((packed)) copy_get;
		struct {
			__le64 snapid;
			__le64 src_version;
			__u8 flags; /* CEPH_OSD_COPY_FROM_FLAG_* */
			/*
			 * CEPH_OSD_OP_FLAG_FADVISE_*: fadvise flags
			 * for src object, flags for dest object are in
			 * ceph_osd_op::flags.
			 */
			__le32 src_fadvise_flags;
		} __attribute__ ((packed)) copy_from;
		struct {
			struct ceph_timespec stamp;
		} __attribute__ ((packed)) hit_set_get;
		struct {
			__u8 flags;
		} __attribute__ ((packed)) tmap2omap;
		struct {
			__le64 expected_object_size;
			__le64 expected_write_size;
			__le32 flags;  /* CEPH_OSD_OP_ALLOC_HINT_FLAG_* */
		} __attribute__ ((packed)) alloc_hint;
		struct {
			__le64 offset;
			__le64 length;
			__le64 data_length;
		} __attribute__ ((packed)) writesame;
		struct {
			__le64 offset;
			__le64 length;
			__le32 chunk_size;
			__u8 type;              /* CEPH_OSD_CHECKSUM_OP_TYPE_* */
		} __attribute__ ((packed)) checksum;
	} __attribute__ ((packed));
	__le32 payload_len;
} __attribute__ ((packed));


/* src/include/object.h */

/** short for stripped object */
struct sobject_t {
    object_t oid;   // { std::string name; }
    snapid_t snap;  // { uint64_t val; }

    /* ... */
};  /* struct sobject_t */
```

```c++
/* src/osdc/Objecter.cc */
void Objecter::_op_submit(Op *op, ceph::shunique_lock<ceph::shared_mutex>& lc,
                            ceph_tid_t *ptid);

/* src/osd/PrimaryLogPG.cc */
int PrimaryLogPG::do_osd_ops(OpContext *ctx, std::vector<OSDOp>& ops);
```

`ObjectOperation` and `Op` are virtually the same in terms of data fields.
`ObjectOperation` only fills parameters of operations, it is generally a helper
class used to generate a transactional operation that contains multiple ops,
while `Op` makes the final `op_submit()` call. E.g.

```c++
/* src/mds/CDir.cc/CDir::_omap_fetch_more() */
  ObjectOperation rd;
  rd.omap_get_vals(...);
  mdcache->mds->objecter->read(oid, oloc, rd, CEPH_NOSNAP, NULL, 0,
                                new C_OnFinisher(fin, mdcache->mds->finisher));
/* results in omap_get_vals + read, atomic & consistent */
```

bufferlist-based RPC Format
---------------------------

> Snapshot related info may be ambiguous or incorrect.

| Variable | Type            |
|----------|-----------------|
| `o`      | `Objecter::Op*` |
| `osd_op` | `OSDOp&`        |

* `CEPH_OSD_OP_STAT`
    * context
        * `o->snapid` (Self-managed snap)
            > AFAIK the per-object self-managed snap, is not used by CephFS.

            > `CEPH_NOSNAP` is `((__u64)-2)`, i.e. 18446744073709551614.
        * `o->target.base_oloc.pool` (omitted later)
        * `o->target.base_oid` (omitted later)
    * return data via `osd_op.outdata` (omitted later)
        1. `uint64_t size`
        2. `utime_t mtime`
    * return value `osd_op.rval` (omitted later)
        * `0` on success
        * `-ENOENT` on not exist

* `CEPH_OSD_OP_CREATE`
    * context
        * `o->mtime`
        * `o->snapc`
    * parameters
        * `osd_op.flags`
        * `osd_op.soid`
        * `osd_op.indata`
            1. `string category` (not implemented and not used)
    * return data
        * none
    * return value
        * `0` on success
        * `-EEXIST` on object already present and `CEPH_OSD_OP_FLAG_EXCL`
            exclusive create flag is set
        * `-EINVAL` on decoding error, currently unreachable

* `CEPH_OSD_OP_DELETE` (namely `Objecter::ObjectOperation::remove()`)
    * context
        * `o->mtime`
        * `o->snapc`
    * parameters
        * `osd_op.soid`
    * return data
        * none
    * return value
        * `0` on success
        * `-ENOENT` on object does not exist

* `CEPH_OSD_OP_READ`
    * context
        * `o->snapid`
    * parameters
        * `osd_op.extent`
            * `.offset`
            * `.length` if 0 read the whole object
            * `.truncate_size`
            * `.truncate_seq`
    * return data
        1. object content
    * return value
        * `0` on success
        * `-ENOENT` on object not exist
        * `-EIO` on CRC mismatch and not `CEPH_OSD_OP_FLAG_FAILOK`
        * or whatever lower level throws, `-EAGAIN` and such
    * return successful read length via `osd_op.extent.length`

* `CEPH_OSD_OP_WRITE`
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset`
            * `.length == osd_op.indata.length()`
            * `.truncate_size`
            * `.truncate_seq`
        * `osd_op.flags`
            * `CEPH_OSD_OP_FLAG_FADVISE_DONTNEED`
        * `osd_op.indata`
            1. new content
    * return data
    * return value
        * `0` on success
        * `-EINVAL` invalid parameter, e.g. when `osd_op.extent.length != osd_op.indata.length()`
        * `-EOPNOSUPP` on unaligned append when `pool.info.requires_aligned_append()` is true
        * `-EFBIG` exceeding `osd_max_object_size`

    > Write may create object silently.

* `CEPH_OSD_OP_WRITEFULL`
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset`
            * `.length == osd_op.indata.length()`
        * `osd_op.flags`
        * `osd_op.indata`
            1. new content
    * return value
        * `0` on success
        * `-EINVAL` invalid parameter, e.g. when `osd_op.extent.length != osd_op.indata.length()`

    > Truncate object to `osd_op.length` if already exists.

* `CEPH_OSD_OP_OMAPSETHEADER`
    * context (from `Objecter::mutate()`)
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset == 0`
            * `.length == osd_op.indata.length()`
        * `osd_op.indata`
            1. header content
    * return value
        * `0` on success
        * `-EOPNOTSUPP` on pool does not support omap

    > May create object silently.

* `CEPH_OSD_OP_OMAPSETVALS`
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset == 0`
            * `.length == osd_op.indata.length()`
        * `osd_op.indata`
            1. encoded `std::map<string, bufferlist>` or `boost::container::flat_map`
                key-value pairs
            > Use `src/os/Transaction.cc/decode_str_str_map_to_bl()` to first extract
            > effective segment in the overall bl. Then decode the extracted
            > proportion entirely.
            >
            > See `src/osd/PrimaryLogPG.cc/PrimaryLogPG::do_osd_ops()`
            > `case CEPH_OSD_OP_OMAPSETVALS`.
    * return value
        * `0` on success
        * `-EINVAL` on decode error
        * `-EOPNOTSUPP` on pool does not support omap

    > May create object silently.

* `CEPH_OSD_OP_OMAPRMKEYS`
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset == 0`
            * `.length == osd_op.indata.length()`
        * `osd_op.indata`
            1. encoded `std::set<string>` or `boost::container::flat_set`
            > `src/os/Transaction.cc/decode_str_set_to_bl()`
    * return value
        * `0` on success
        * `-ENOENT` on object not exist
        * `-EINVAL` on parameter decode error
        * `-EOPNOTSUPP` on pool does not support omap

> Below are operations not so common for MDS services.

* `CEPH_OSD_OP_OMAPGETHEADER`
    * context (from `Objecter::read()`)
        * `o->snapid`
    * parameters
        * none
    * return data
        1. header content
    * return value
        * `0` on success
        * `-ENOENT` on object not exist

* `CEPH_OSD_OP_OMAPGETVALS`
    * context (from `Objecter::read()`)
        * `o->snapid`
    * parameters
        * `osd_op.extent`
            * `.offset == 0`
            * `.length == osd_op.indata.length()`
        * `osd_op.indata`
            1. `std::string start_after`
            2. `uint64_t max_to_get`
            3. `sdt::string filter_prefix`
    * return data
        1. `uint32_t num` total number of retrieved KV pairs
        2. `std::string key1`
        3. `ceph::buffer::list value1`
        4. `std::string key2`
        5. `ceph::buffer::list value2`
        6. ... more keys
        7. ... more values
        9. `bool truncated` if return data is truncated due to reaching `max_to_get` or `_conf->osd_max_omap_bytes_per_request`

        > 1 ~ 7 can be decoded with `ceph::decode()` overloaded for `std::map<string, bufferlist>`
        >
        > See also `src/osdc/Objecter.h/ObjectOperation/CB_ObjectOperation_decodevals()`
    * return value
        * `0` on success
        * `-EINVAL` on parameter decode error
        * `-ENOENT` on object does not exist

* `CEPH_OSD_OP_OMAPGETVALSBYKEY`
    * context (from `Objecter::read()`)
        * `o->snapid`
    * parameters
        * `osd_op.extent`
            * `.offset == 0`
            * `.length == osd_op.indata.length()`
        * `osd_op.indata`
            1. `std::set<std::string>` or `boost::container::flat_set<std::string>` of keys
    * return data
        1. `std::map<std::string, bufferlist>` of kv pairs
    * return value
        * `0` on success
        * `-EINVAL` on parameter decode error
        * `-ENOENT` on object does not exist

* `CEPH_OSD_OP_ZERO`
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset`
            * `.length`
    * return value
        * `0` on success
        * `-EOPNOSUPP` if pool `requires_aligned_append()`
        * `-EFBIG` exceeding `osd_max_object_size`

    > No-op if object does not exist.

* `CEPH_OSD_OP_OMAPRMKEYS`
    * context (from `Objecter::mutate()`)
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.indata`
            1. `std::set<std::string>` or `boost::container::flat_set<std::string>` of keys
    * return value
        * `0` on success
        * `-ENOSUPPORT` on pool does not support omap
        * `-ENOENT` on object not exist
        * `-EINVAL` on parameter decode error

* `CEPH_OSD_OP_OMAPCLAER`
    * context (from `Objecter::mutate()`)
        * `o->snapc`
        * `o->mtime`
    * return value
        * `0` on success
        * `-EOPNOSUPP` if pool does not support omap
        * `-ENOENT` if object does not exist

* `CEPH_OSD_OP_SETXATTR`
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.xattr`
            * `.name_len == strlen(name) or 0 if not provided`
            * `.value_len == bl.length()`
        * `osd_op.indata`
            1. `char *name` if provided (trailing `'\0'` not included)
            2. `ceph::buffer::list bl` value
    * return value
        * `0` on success
        * `-EFBIG` if value larger than `_conf->osd_max_attr_size` (if set)
        * `-ENAMETOOLONG` if name longer than either `osd->store->get_max_attr_name_length()` or `_conf->osd_max_attr_name_len`

    > May silently create object if not exist.

    > In original Ceph, the on-disk name for an xattr is `"_<name>"` therefore it is okay to not provide xattr name (the corresponding key in DB would be `"_"`).

* `CEPH_OSD_OP_RMXATTR` (namely `Objecter::removexattr()`)
    * context
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.xattr`
            * `.name_len == strlen(name) or 0 if not provided`
            * `.value_len == bl.length()`
        * `osd_op.indata`
            1. `char *name` if provided
    * return value
        * `0` on success
        * `-ENOENT` if object does not exist

* `CEPH_OSD_OP_GETXATTR`
    * context
        * `o->snapid`
        * `o->mtime`
    * parameters
        * `osd_op.xattr`
            * `.name_len == strlen(name) or 0 if not provided`
            * `.value_len == 0`
        * `osd_op.indata`
            1. `char *name` if provided
    * return data
        1. xattr value
    * return value
        * `0` on success
        * `-ENOENT` on object not exist
        * `-ENODATA` on xattr not exist
    * return value length via `osd_op.xattr.value_len`

* `CEPH_OSD_OP_TRUNCATE` (namely `ObjectOperation::truncate()`)
    * context (from `Objecter::mutate()`)
        * `o->snapc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.offset`
            * `.length == 0`
        * `osd_op.indata`
            1. empty `ceph::buffer::list`
    * return value
        * `0` on success
        * `-EOPNOSUPP` if pool `requires_aligned_append()`
        * `-EFBIG` if exceeding `osd_max_object_size`

    > No-op if object does not exist.

* `CEPH_OSD_OP_TRIMTRUNC` (via `src/osdc/Filer.cc/Filer::truncate()`)
    * context (from `Objecter::_modify()`)
        * `o->spanc`
        * `o->mtime`
    * parameters
        * `osd_op.extent`
            * `.truncate_seq`
                > Truncate sequence seems to be just a sequence number of
                > asynchronous truncation jobs.
                >
                > See also `src/mds/mdstypes.h/struct inode_t`. For every call to
                > `inode_t::truncate()`, the sequence number increases.
                > `PrimaryLogPG` treats incoming truncation with no larger seq as
                > no-op.
            * `.truncate_size` same as `.offset` with `CEPH_OSD_OP_TRUNCATE`


About Transaction
-----------------

After the `Op` is constructed on `osdc` (OSD Client) side, the `Op` is sent to
the remote OSD server for the `OSDOp`s in `Op` to be actually executed.

For read ops, they are eager executed in `PrimaryLogPG::do_osd_ops()` (this
function sits in `PrimaryLogPG::prepare_transaction()` procedure, which makes its
naming __really really confusing__, for it only does / performs part of the
received ops, not all of them). For writes and modifications, on object's blob
or xattrs or OMAP, they are pushed into `PGTransaction` to be later submitted to
`PGBackend`.

`OSDOp`s of aborted transactions are returned as is, that is, OSD may return
vector of `OSDOp`s that has the first few ops done, the one somewhere in the
middle failed and the trailing untouched (default-initialized). The successful
ops will have `rval` of 0, the failed one will have negative `rval`, and their
handlers are expected to skip decoding for these ops. The untouched ones will have
untouched `rval`s, which will probably (behavior defined by handler implementation)
be zero, their decoding handler will get executed, but will no doubt throw
`ceph::buffer::error` and then modify its `rval = -EIO`.

The supplied callback (i.e. `onfinish` in `Objecter::handle_osd_op_reply()`),
however, takes the overall return code (i.e. `rc` in `Objecter::handle_osd_op_reply()`,
which is the `result` return of `PrimaryLogPG::do_osd_ops()`, the return code of
the failed and not ignored sub-op in the transactional `Op`). Typically (behavior
defined by implementation) does nothing or specific error handling if it is not
zero or positive.
