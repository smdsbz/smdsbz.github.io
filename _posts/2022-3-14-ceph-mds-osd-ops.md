---
layout: article
title: Ceph MDS OSD Ops Trace
key: ceph-mds-osd-ops-trace
tags: Storage DistributedStorage Ceph SourceCode MDS
---

<!-- more -->

## Parsing Trace


```python
import re
from typing import Any

__log_regex = re.compile(
    r'\[(?P<time>[\d:\.]+)\] \((?P<time_offset>\+[\d\.]+)\) (?P<host>\S+) '
    r'(?P<log_module>\S+):(?P<log_label>\S+): \{ cpu_id = (?P<cpu_id>\d+) \}, '
    r'\{ (?P<params>.*) \}'
)

def parse_trace(fname: str) -> list[dict[str, Any]]:
    trace: list[dict[str, Any]] = []
    with open(fname, 'rt') as f:
        for l in f:
            while l[-1] in '\r\n':
                l = l[:-1]
            m = __log_regex.match(l)
            if not m:
                raise RuntimeError(f'failed to parse line \"{l}\"')
            log_entry = m.groupdict()
            params_raw = log_entry['params']
            log_entry['params'] = {}
            for kv in params_raw.split(','):
                k, v = kv.split('=')
                k = k.strip()
                v = eval(v)
                log_entry['params'][k.strip()] = v
            trace.append(log_entry)
    return trace

full_trace = parse_trace('./mds_trace1.log')
```


```python
for l in full_trace[:10]:
    print(l)
```

    {'time': '18:31:14.755314381', 'time_offset': '+0.000006225', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre', 'cpu_id': '0', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'op': 8705, 'opname': 'write', 'flags': 32}}
    {'time': '18:31:14.755319721', 'time_offset': '+0.000005340', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre_write', 'cpu_id': '0', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'osize': 2225, 'oseq': 0, 'offset': 2225, 'length': 2889, 'truncate_size': 0, 'truncate_seq': 0}}
    {'time': '18:31:14.755336030', 'time_offset': '+0.000016309', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_post', 'cpu_id': '0', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'op': 8705, 'opname': 'write', 'flags': 32, 'result': 0}}
    {'time': '18:31:16.349494063', 'time_offset': '+0.000007095', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre', 'cpu_id': '2', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'op': 8705, 'opname': 'write', 'flags': 32}}
    {'time': '18:31:16.349498528', 'time_offset': '+0.000004465', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre_write', 'cpu_id': '2', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'osize': 5114, 'oseq': 0, 'offset': 5114, 'length': 3158740, 'truncate_size': 0, 'truncate_seq': 0}}
    {'time': '18:31:16.349513796', 'time_offset': '+0.000015268', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_post', 'cpu_id': '2', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'op': 8705, 'opname': 'write', 'flags': 32, 'result': 0}}
    {'time': '18:31:17.350160628', 'time_offset': '+0.000007759', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre', 'cpu_id': '1', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'op': 8705, 'opname': 'write', 'flags': 32}}
    {'time': '18:31:17.350165804', 'time_offset': '+0.000005176', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre_write', 'cpu_id': '1', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'osize': 3163854, 'oseq': 0, 'offset': 3163854, 'length': 1030450, 'truncate_size': 0, 'truncate_seq': 0}}
    {'time': '18:31:17.350182950', 'time_offset': '+0.000017146', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_post', 'cpu_id': '1', 'params': {'oid': '200.00000001', 'snap': 18446744073709551614, 'op': 8705, 'opname': 'write', 'flags': 32, 'result': 0}}
    {'time': '18:31:17.353091345', 'time_offset': '+0.000008082', 'host': 'yyx-XPS-13-9365', 'log_module': 'osd', 'log_label': 'do_osd_op_pre', 'cpu_id': '3', 'params': {'oid': '200.00000007', 'snap': 18446744073709551614, 'op': 8709, 'opname': 'delete', 'flags': 0}}
    


```python
from collections import defaultdict

pre_ops = []
param_ops = []
post_ops = []
for t in full_trace:
    if t['log_label'] == 'do_osd_op_pre':
        pre_ops.append(t)
    elif t['log_label'].startswith('do_osd_op_pre_'):
        param_ops.append(t)
    elif t['log_label'] == 'do_osd_op_post':
        post_ops.append(t)
    else:
        raise RuntimeError(f'unclassifiable label: {t}')

param_ops_by_opname = defaultdict(list)
for t in param_ops:
    param_ops_by_opname[t['log_label'][len('do_osd_op_pre_'):]].append(t)
```


```python
for opname, ops in param_ops_by_opname.items():
    print(opname, '\t', 'occurance:', len(ops))
    for op in ops[:3]:
        print(op['params'])
```

    write 	 occurance: 12296
    {'oid': '200.00000001', 'snap': 18446744073709551614, 'osize': 2225, 'oseq': 0, 'offset': 2225, 'length': 2889, 'truncate_size': 0, 'truncate_seq': 0}
    {'oid': '200.00000001', 'snap': 18446744073709551614, 'osize': 5114, 'oseq': 0, 'offset': 5114, 'length': 3158740, 'truncate_size': 0, 'truncate_seq': 0}
    {'oid': '200.00000001', 'snap': 18446744073709551614, 'osize': 3163854, 'oseq': 0, 'offset': 3163854, 'length': 1030450, 'truncate_size': 0, 'truncate_seq': 0}
    delete 	 occurance: 19
    {'oid': '200.00000007', 'snap': 18446744073709551614}
    {'oid': '200.00000008', 'snap': 18446744073709551614}
    {'oid': '200.00000009', 'snap': 18446744073709551614}
    omapsetheader 	 occurance: 30
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    omapsetvals 	 occurance: 27
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    writefull 	 occurance: 10
    {'oid': '200.00000000', 'snap': 18446744073709551614, 'osize': 90, 'offset': 0, 'length': 90}
    {'oid': '200.00000000', 'snap': 18446744073709551614, 'osize': 90, 'offset': 0, 'length': 90}
    {'oid': '200.00000000', 'snap': 18446744073709551614, 'osize': 90, 'offset': 0, 'length': 90}
    create 	 occurance: 1
    {'oid': '10000001976.00000000', 'snap': 18446744073709551614}
    omaprmkeys 	 occurance: 25
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    {'oid': 'mds0_openfiles.0', 'snap': 18446744073709551614}
    read 	 occurance: 751
    {'oid': '100000027ee.00000000', 'snap': 18446744073709551614, 'osize': 131, 'oseq': 0, 'offset': 0, 'length': 131, 'truncate_size': 0, 'truncate_seq': 0}
    {'oid': '10000002feb.00000000', 'snap': 18446744073709551614, 'osize': 578, 'oseq': 0, 'offset': 0, 'length': 578, 'truncate_size': 0, 'truncate_seq': 0}
    {'oid': '10000001c2e.00000000', 'snap': 18446744073709551614, 'osize': 982, 'oseq': 0, 'offset': 0, 'length': 982, 'truncate_size': 0, 'truncate_seq': 0}
    

## Analysis

### OSD Operations


```python
print('all ops occurred:', param_ops_by_opname.keys())
```

    all ops occurred: dict_keys(['write', 'delete', 'omapsetheader', 'omapsetvals', 'writefull', 'create', 'omaprmkeys', 'read'])
    

There could be other ops that are invoked by MDS, but not listed above.

### Snaps


```python
import struct

def u64_to_i64(u: int) -> int:
    # HACK: big-endian, for hex string is in human-readable order
    return struct.unpack('>q', bytes.fromhex(hex(u)[2:]))[0]
def u32_to_i64(u: int) -> int:
    return struct.unpack('>i', bytes.fromhex(hex(u)[2:]))[0]

print('all snaps seen:', {u64_to_i64(t['params']['snap']) for t in full_trace if 'snap' in t['params']})
```

    all snaps seen: {-2}
    

Where snap ID -2 is `CEPH_NOSNAP`.

> Such conclusion is drawn from a trace that does not include snapshots.

### Create


```python
create_params = set()
for t in param_ops_by_opname['create']:
    create_params.update(t['params'].keys())
print('create parameters:', create_params)
```

    create parameters: {'oid', 'snap'}
    


```python
print('all create op flags:', {hex(t['params']['flags']) for t in post_ops if t['params']['opname'] == 'create'})
print('all create op result:', {hex(t['params']['result']) for t in post_ops if t['params']['opname'] == 'create'})
```

    all create op flags: {'0x1'}
    all create op result: {'0x0'}
    

`src/include/rados.h`

```
	CEPH_OSD_OP_FLAG_EXCL = 0x1,      /* EXCL object create */
```

### Delete


```python
delete_params = set()
for t in param_ops_by_opname['delete']:
    delete_params.update(t['params'].keys())
print('delete parameters:', delete_params)
```

    delete parameters: {'oid', 'snap'}
    


```python
print('all delete op flags:', {hex(t['params']['flags']) for t in post_ops if t['params']['opname'] == 'delete'})
print('all delete op result:', {u32_to_i64(t['params']['result']) for t in post_ops if t['params']['opname'] == 'delete'})
```

    all delete op flags: {'0x0'}
    all delete op result: {-2}
    

`/usr/include/asm-generic/errno-base.h`

```
#define	ENOENT		 2	/* No such file or directory */
```

### Read


```python
read_params = set()
for t in param_ops_by_opname['read']:
    read_params.update(t['params'].keys())
print('read parameters:', read_params)
```

    read parameters: {'oid', 'offset', 'length', 'truncate_size', 'osize', 'truncate_seq', 'oseq', 'snap'}
    


```python
for p in ('oseq', 'truncate_seq', 'truncate_size'):
    print(f'all {p} seen:', {t['params'][p] for t in param_ops_by_opname['read']})
```

    all oseq seen: {0}
    all truncate_seq seen: {0}
    all truncate_size seen: {0}
    


```python
print('all read op flags:', {hex(t['params']['flags']) for t in post_ops if t['params']['opname'] == 'read'})
print('all read op result:', {hex(t['params']['result']) for t in post_ops if t['params']['opname'] == 'read'})
```

    all read op flags: {'0x0'}
    all read op result: {'0x0'}
    

### Write


```python
write_params = set()
for t in param_ops_by_opname['write']:
    write_params.update(t['params'].keys())
print('write parameters:', write_params)
```

    write parameters: {'oid', 'offset', 'length', 'truncate_size', 'osize', 'truncate_seq', 'oseq', 'snap'}
    


```python
for p in ('oseq', 'truncate_seq', 'truncate_size'):
    print(f'all {p} seen:', {t['params'][p] for t in param_ops_by_opname['write']})
```

    all oseq seen: {0}
    all truncate_seq seen: {0}
    all truncate_size seen: {0}
    


```python
print('all write op flags:', {hex(t['params']['flags']) for t in post_ops if t['params']['opname'] == 'write'})
print('all write op result:', {hex(t['params']['result']) for t in post_ops if t['params']['opname'] == 'write'})
```

    all write op flags: {'0x20', '0x0'}
    all write op result: {'0x0'}
    

`src/include/rados.h`

```
	CEPH_OSD_OP_FLAG_FADVISE_DONTNEED   = 0x20,/* data will not be accessed in the near future */
```


```python
print('writes with 0x00 flag', len([None for t in post_ops if t['params']['opname'] == 'write' and t['params']['flags'] == 0x00]))
print('writes with 0x20 flag', len([None for t in post_ops if t['params']['opname'] == 'write' and t['params']['flags'] == 0x20]))
```

    writes with 0x00 flag 10751
    writes with 0x20 flag 1545
    

### WriteFull


```python
writefull_params = set()
for t in param_ops_by_opname['writefull']:
    writefull_params.update(t['params'].keys())
print('writefull parameters:', writefull_params)
```

    writefull parameters: {'oid', 'offset', 'length', 'osize', 'snap'}
    


```python
for p in ('offset', 'oid', 'osize', 'length'):
    print(f'all {p} seen:', {t['params'][p] for t in param_ops_by_opname['writefull']})
```

    all offset seen: {0}
    all oid seen: {'200.00000000'}
    all osize seen: {90}
    all length seen: {90}
    


```python
print(all((t['params']['offset'] == 0 for t in param_ops_by_opname['writefull'])))
print(all((t['params']['osize'] == t['params']['length'] for t in param_ops_by_opname['writefull'])))
```

    True
    True
    


```python
print('all writefull op flags:', {hex(t['params']['flags']) for t in post_ops if t['params']['opname'] == 'writefull'})
print('all writefull op result:', {hex(t['params']['result']) for t in post_ops if t['params']['opname'] == 'writefull'})
```

    all writefull op flags: {'0x20'}
    all writefull op result: {'0x0'}
    

`src/include/rados.h`

```
	CEPH_OSD_OP_FLAG_FADVISE_DONTNEED   = 0x20,/* data will not be accessed in the near future */
```

__TODO:__ Judging by its name, since we're overwriting the entire object, object size `osize` should be
equal to expected size `length`, but find proof in source code just to be sure.

### Omap-related


```python
omapsetheader_params = set()
for t in param_ops_by_opname['omapsetheader']:
    omapsetheader_params.update(t['params'].keys())
print('omapsetheader parameters:', omapsetheader_params)
```

    omapsetheader parameters: {'oid', 'snap'}
    


```python
print('all oid seen:', {t['params']['oid'] for t in param_ops_by_opname['omapsetheader']})
```

    all oid seen: {'mds0_openfiles.0'}
    


```python
omapsetvals_params = set()
for t in param_ops_by_opname['omapsetvals']:
    omapsetvals_params.update(t['params'].keys())
print('omapsetvals parameters:', omapsetvals_params)
```

    omapsetvals parameters: {'oid', 'snap'}
    


```python
print('all oid seen:', {t['params']['oid'] for t in param_ops_by_opname['omapsetvals']})
```

    all oid seen: {'mds0_openfiles.0'}
    


```python
omaprmkeys_params = set()
for t in param_ops_by_opname['omaprmkeys']:
    omaprmkeys_params.update(t['params'].keys())
print('omaprmkeys parameters:', omaprmkeys_params)
```

    omaprmkeys parameters: {'oid', 'snap'}
    


```python
print('all oid seen:', {t['params']['oid'] for t in param_ops_by_opname['omaprmkeys']})
```

    all oid seen: {'mds0_openfiles.0'}
    


```python
print('all omap op flags:', {hex(t['params']['flags']) for t in post_ops if t['params']['opname'].startswith('omap')})
print('all omap op result:', {hex(t['params']['result']) for t in post_ops if t['params']['opname'].startswith('omap')})
```

    all omap op flags: {'0x0'}
    all omap op result: {'0x0'}
    
