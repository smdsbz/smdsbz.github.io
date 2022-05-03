---
layout: article
title: Ceph Objecter bufferlist Format
key: ceph-objecter-bufferlist
tags: Storage DistributedStorage Ceph SourceCode MDS
---

How calling parameters and return data are encoded in Ceph Objecter RPC.

<!-- more -->

Code Teardown Reminder
----------------------

* Encoding procedure can be found at `src\osdc\Objecter.h\Objecter::prepare_<op>_op()`.
* Decoding procedure can be referenced at `src\osdc\Objecter.h\ObjectOperation::struct CB_ObjectOperation_<op>::operator()()` or `src\osdc\Objecter.h\Objecter::C_<Op>::finish()`

Key Data Structures
-------------------

```c++
namespace Objecter {

/******** Packaged OSD Operations ********/

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
struct Op {
    /* ... */

    osdc_opvec ops;     // OSDOp[]

    snapid_t snapid = CEPH_NOSNAP;
    SnapContext snapc;
    ceph::real_time mtime;

    /* final output of the entire sequence of ops, usually organized and filled
        by callback */
    ceph::buffer::list *outbl = nullptr;
    /* individual output of each member op, which are just pointers to underlying
        fields of individual ops. zeroed by default, same size as `ops` */
    boost::container::small_vector<ceph::buffer::list*, osdc_opvec_len> out_bl;
    boost::container::small_vector<
        fu2::unique_function<void(boost::system::error_code, int,
                                    const ceph::buffer::list& bl) &&>,
        osdc_opvec_len> out_handler;
    boost::container::small_vector<int*, osdc_opvec_len> out_rval;
    boost::container::small_vector<boost::system::error_code*,
                                    osdc_opvec_len> out_ec;

    /* ... */
};  /* struct Op */


/******** Individual OSD Operation ********/

struct ceph_osd_op;
struct sobject_t;

/* src/osd/osd_types.h */
struct OSDOp {
    ceph_osd_op op;     // opcode, flag and extra calling args
    sobject_t soid;

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
struct sobject_t {
    object_t oid;   // { std::string name; }
    snapid_t snap;  // { uint64_t val; }

    /* ... */
};  /* struct sobject_t */

}   /* namespace Objecter */
```

```c++
/* src/osdc/Objecter.cc */
void Objecter::_op_submit(Op *op, ceph::shunique_lock<ceph::shared_mutex>& lc,
                            ceph_tid_t *ptid);

/* src/osd/PrimaryLogPG.cc */
int PrimaryLogPG::do_osd_ops(OpContext *ctx, std::vector<OSDOp>& ops);
```

`ObjectOperation` and `Op` are virtually the same in terms of data fields.
`ObjectOperation` only fills parameters of operations (usually used to generate
a transactional operation that contains multiple ops), while `Op` finally makes
the `op_submit()` call. E.g.

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

* `CEPH_OSD_OP_STAT`
    * context
        * `op->snapid` (I/O context)
            > AFAIK the per-object self-managed snap, i.e. `osd_op.soid.snap.val`,
            > is not used by CephFS.

            > `CEPH_NOSNAP` is `((__u64)-2)`.
    * parameters
        * `osd_op.soid` (TODO: conversion to `hobject_t`, with pool name and such)
    * return via `osd_op->outdata`
        * `uint64_t size`
        * `utime_t mtime`
    * `osd_op.rval`
        * `0` on success
        * `-ENOENT` on not exist
