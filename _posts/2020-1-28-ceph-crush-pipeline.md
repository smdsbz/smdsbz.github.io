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

**Algorithm** CRUSH placement for object $x$

$$
\begin{aligned}
1:\  &\hspace{0em}  \textbf{procedure} \ \text{TAKE}(\alpha) &\text{//将项}\ \alpha\ \text{放入工作列表}\ \vec{i}\ \text{中} \\
2:\  &\hspace{2em}  \vec{i} \leftarrow \alpha \\
3:\  &\hspace{0em}  \textbf{end procedure} \\
\\
4:\  &\hspace{0em}  \textbf{procedure} \ \text{SELECT}(n,t) &\text{//选择}\ n\ \text{个类型为}\ t\ \text{的项目} \\
5:\  &\hspace{2em}  \vec{o} \leftarrow \emptyset &\text{//初始化输出变量为空} \\
6:\  &\hspace{2em}  \textbf{for}\ i \in \vec{i}\ \textbf{do} &\text{//遍历输入变量}\ \vec{i} \\
7:\  &\hspace{4em}  f \leftarrow 0 &\text{//目前没有操作失败} \\
8:\  &\hspace{4em}  \textbf{for}\ r \leftarrow 1, n\ \textbf{do} &\text{//遍历}\ n\ \text{个副本} \\
9:\  &\hspace{6em}  f_r \leftarrow 0 &\text{//该副本上操作还未遇到失败} \\
10:\ &\hspace{6em}  retry\_descent \leftarrow false \\
11:\ &\hspace{6em}  \textbf{repeat} \\
12:\ &\hspace{8em}  b \leftarrow bucket(i) &\text{//从桶}\ i\ \text{开始递归} \\
13:\ &\hspace{8em}  retry\_bucket \leftarrow false \\
14:\ &\hspace{8em}  \textbf{repeat} \\
15:\ &\hspace{10em} \textbf{if}\ \text{``first n"}\ \textbf{then} &\text{//见 3.2.2 节} \\
16:\ &\hspace{12em} r' \leftarrow r + f \\
17:\ &\hspace{10em} \textbf{else} \\
18:\ &\hspace{12em} r' \leftarrow r + f_r n \\
19:\ &\hspace{10em} \textbf{end if} \\
20:\ &\hspace{10em} o \leftarrow b.c(r',x) &\text{//见 3.4 节} \\
21:\ &\hspace{10em} \textbf{if}\ type(o) \neq t\ \textbf{then} \\
22:\ &\hspace{12em} b \leftarrow bucket(o) &\text{//继续递归} \\
23:\ &\hspace{12em} retry\_bucket \leftarrow true \\
24:\ &\hspace{10em} \textbf{else if}\ o \in \vec{o}\ \text{or}\ failed(o)\ \text{or}\ overload(o,x)\ \textbf{then} \\
25:\ &\hspace{12em} f_r \leftarrow f_r + 1, f \leftarrow f + 1 \\
26:\ &\hspace{12em} \textbf{if}\ o \in \vec{o}\ \text{and}\ f_r \lt 3\ \textbf{then} \\
     &\hspace{14em} \text{//在当前桶内重试以解决冲突（见 3.2.1 节）} \\
27:\ &\hspace{14em} retry\_bucket \leftarrow true \\
28:\ &\hspace{12em} \textbf{else} \\
29:\ &\hspace{14em} retry\_descent \leftarrow true &\text{//否则从}\ i\ \text{开始重新递归} \\
30:\ &\hspace{12em} \textbf{end if} \\
31:\ &\hspace{10em} \textbf{end if} \\
32:\ &\hspace{8em}  \textbf{until}\ \neg retry\_bucket \\
33:\ &\hspace{6em}  \textbf{until}\ \neg retry\_descent \\
34:\ &\hspace{6em}  \vec{o} \leftarrow [\vec{o},o] &\text{//将}\ o\ \text{添加到输出}\ \vec{o}\ \text{中} \\
35:\ &\hspace{4em}  \textbf{end for} \\
36:\ &\hspace{2em}  \textbf{end for} \\
37:\ &\hspace{0em}  \vec{i} \leftarrow \vec{o} &\text{//将输出拷贝回}\ \vec{i}\ \text{中} \\
38:\ &\hspace{0em}  \textbf{end procedure} \\
\\
39:\ &\hspace{0em}  \textbf{procedure}\ \text{EMIT} &\text{//将工作列表}\ \vec{i}\ \text{追加到结果中} \\
40:\ &\hspace{2em}  \vec{R} \leftarrow [\vec{R},\vec{i}] \\
41:\ &\hspace{0em}  \textbf{end procedure} \\
\end{aligned}
$$


$$
\begin{aligned}
E(d_\text{cached})
&= p_\text{hit} \big( E(d_\text{ssd}) + E(d_\text{network}) \big) \\
&+ (1 - p_\text{hit}) \underbrace{\big( E(d_\text{hdd}) + E_{c,\text{ssd}}(d_\text{dist}) \big)}_\text{promote lat.} \\
&+ (1 - p_\text{hit}) (1 - p_\text{read}) \underbrace{\big( E(d_\text{ssd}) + E_{b,\text{hdd}}(d_\text{dist}) \big)}_\text{flush lat.} \\
&+ d_\text{software} \\
\\
E_{r,\text{type}} (d_\text{dist})
&= E(\max_r d_\text{network}) + E(\max_r d_\text{type}) \hspace{1em} \text{s.t. type} \in \{\text{ssd}, \text{hdd}\} \\
\end{aligned}
$$

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

    Spits out empty active set when OSD mapping is stable (for `temp_*` members
    will not be specified then), and `OSDMap:_pg_to_raw_osds()` gets executed.

    > `temp_osd` stands for OSDs that are temporarily responsible and currently
    > __available__ for incoming I/O.
    >
    > See [here](https://blog.csdn.net/wojiaowugen/article/details/80204121) for
    > why the `temp` mechanism is needed.

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

    `src/osd/OSDMap.h/OSDMap::_pg_to_raw_osds()`

    Invokes `CrushWrapper::do_rule()`.

    > To mess with the default PG-to-OSD placement policy, we may make our
    > changes in this function.

    ```c++
      /// pg -> (raw osd list)
      void _pg_to_raw_osds(
        const pg_pool_t& pool, pg_t pg,
        vector<int> *osds,
        ps_t *ppps) const;
    ```
