---
layout: article
title: Ceph Objecter op_submit()
key: ceph-objecter-op-submit
tags: Storage DistributedStorage Ceph SourceCode
---

<!-- more -->

_Based on Ceph v16.2.5_


Architecture
============

The `Objecter` is how Ceph clients, i.e. _OSDC_, packages all its requests. It
works asynchonously in a callback fashion, where `Context*`, which is just Ceph's
own implementation of `std::function`, are associated with parameterized requests
`Op`. The `Op` is sent to responsible OSD via `Messenger`, and on receiving reply
`Objecter::ms_dispatch()` is kicked off by `Messenger`, where the callbacks will
get executed.


Source
======

* `src/osdc/Objecter.(h|cc)`

Parameterizing
--------------

`ObjectOperation` is initialized, users may call its member functions to stuff
requests into it.

Handling
--------

|![](https://images2015.cnblogs.com/blog/971979/201606/971979-20160609222516183-1221469378.png)|
|:-:|
| OSDC sending request (exemplified by `rados_write_full()`) |

* `op_submit()`
    * take `Objecter` lock
    * `_op_submit_with_budget()`
        * throttling?
        * set timeout callback
    * `_op_submit()`
        * calculate target OSDs, and get `OSDSession`
        * `_send_op_account()`
            * inflight OSD requests counter `inflight_ops++`
            * pending completions counter `num_in_flight++`
            * performance counters
        * `_session_op_assign()`  
            assign `OSDSession` to `Op`
        * `_send_op()`
            * convert `Objecter` request `Op` to `Messenger` request `MOSDOp`
            * send `MOSDOp` thru `OSDSession`
        * unlock, release `OSDSession`

`Messenger` on OSD gets the request, and processes it.

|![](http://images2015.cnblogs.com/blog/971979/201606/971979-20160609221750090-780839740.jpg)|
|:-:|
| OSD handling request |

`Messenger` on OSDC listening for incomming responds.

Ending
------

__On successfully return__

* `Objecter::ms_dispatch()`  
    _overriding `Dispatcher`_
    * if `CEPH_MSG_OSD_OPREPLY` do `handle_osd_op_reply()`
        * some context validity checking
        * `op->trace.event("osd op reply)` for zipkin trace
        * re-`_op_submit()` if returnted retry / redirect / `-EAGAIN`
        * align return data field `out_(bl/rval/ec)` and call `out_handler`
            * `out_bl` pointers in `Objecter::Op` will be forced to point to
                corresponding received `OSDOp::outdata`
            * `rval` and `ec` will be converted to corresponding host OS error
                codes from received `OSDOp::rval`
            * __`out_handler` will be executed__, all calling parameters are
                provided by the received `OSDOp`
        * `num_in_flight--` if any callback
        * log `l_osdc_op_reply`
        * (get `OSDSession` lock and) `_finish_op()` do callback
        * release `MOSDOpReply`
    * ... (handler for several other types of `Message`s) ...

__On timeout__

* `op_cancel(tid, -ETIMEOUT)`
    * `num_in_flight--` and execute associated callback with `-ETIMEOUT`
    * `_op_cancel_map_check()`
        * erase from `check_latest_map_ops`

        > `Op`s that were waiting for latest OSD map were pushed into this map
        > during `_op_submit()` with `_send_op_map_check()`, if `_calc_target()`
        > determines `check_for_latest_map` is true.

    * `_finish_op()`
        * `put_op_budget_bytes()` accumulate budget to throttler
        * erase from timeout pool
        * `_session_op_remove()` release `OSDSession` ref cntr
        * `inflight_ops--` (and `l_osd_op_active` in logger)
        * release `Op` ref cntr

> The timeout case could be a compact implementation of properly ending and
> releasing an `Op`. We may end up just using `op_cancel(tid, 0)` instead of
> reimplementing everything in `handle_osd_op_reply()`.


- - - - - - - - - - - - - - - - - - - -

* [Ceph源码解析：读写流程](https://www.cnblogs.com/chenxianpao/p/5572859.html)
