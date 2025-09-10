---
layout: article
title: Milvus loading process
key: milvus-load-process
tags: Milvus VectorDatabase
---

Source tear-down of Milvus's loading process.

<!-- more -->

Code paths
==========

Common
------

1. proxy: receive load request and forward to mixcoord

    * internal/proxy/impl.go // (*Proxy).LoadCollection
    * internal/proxy/task.go // (*loadCollectionTask).{PreExecute,Execute}

2. querycoord: submit loading job  

    The loading job is published as a "target", the observer of the target is responsible for maintaining the collection's state and actually trigger segment loading on querynodes.

    * internal/querycoordv2/services.go // (*Server).LoadCollection
    * internal/querycoordv2/job/job_load.go // (*LoadCollectionJob).Execute
    * internal/querycoordv2/observers/target_observer.go // (*TargetObserver).{syncNextTargetToDelegator,syncToDelegator}
        > Syncs the collection's loading state to the delegator querynode, the loading task is executed on the delegator querynode.

3. querynode: load segments locally, or forward to workers if is delegator

    * internal/querynodev2/services.go // (*QueryNode).SyncDistribution
    * internal/querynodev2/delegator/delegator_data.go // (*shardDelegator).LoadSegments
    * internal/querynodev2/{cluster/worker,local_worker}.go // (*Worker).LoadSegments
        > `worker.go` forwards the load request with client, inspecting the local code path `local_worker.go` which actually loads the segments.
    * internal/querynodev2/services.go // (*QueryNode).LoadSegments
    * internal/querynodev2/segments/segment_loader.go // (*segmentLoader).Load
    * internal/querynodev2/segments/segment_loader.go // (*segmentLoader).Load

4. internal/querynodev2/segments/segment_loader.go // (*segmentLoader).Load

    * loader.prepare
    * NewSegment
    * loadSegmentFunc
        * loader.LoadSegment
            > If sealed loader.LoadSealedSegment, otherwise segment.LoadMultiFieldData .
        * loader.loadDeltalogs

Physical load by type
---------------------

### Load field data

Only load when index does not contain raw data (this is true for vector indices with quantization but without refine, see knowhere IndexNode implementations, and scalar indices) and primary key.

Whether the data is loaded as mmap is determined by the mmap configuration.

TODO

* internal/querynodev2/segments/segment.go // (*LocalSegment).LoadFieldData
* internal/core/src/segcore/segment_c.cpp // LoadFieldData

#### Load sealed data

* internal/core/src/segcore/ChunkedSegmentSealedImpl.cpp
    > If not variable length additionally SegmentInterface::LoadSkipIndex .
* internal/core/src/segcore/ChunkedSegmentSealedImpl.cpp // generate_interim_index
    > IVF_FLAT_CC (default) or SCANN_DVR for dense vector, SPARSE_WANT_CC for sparse vector configured with SPARSE_WAND index, SPARSE_INVERTED_INDEX_CC for other sparse vector.
    >
    > "CC" stands for concurrent ([IVF_FLAT_CC](https://github.com/milvus-io/knowhere/pull/824)).

TODO

#### Load growing data

TODO

* internal/core/src/segcore/SegmentGrowingImpl.cpp

### Load sealed index

* internal/querynodev2/segments/segment_loader.go // (*segmentLoader).LoadSealedSegment
* internal/querynodev2/segments/segment_loader.go // (*segmentLoader).loadFieldsIndex
* internal/querynodev2/segments/segment_loader.go // (*segmentLoader).loadFieldIndex
* internal/querynodev2/segments/segment.go // (*LocalSegment).LoadIndex
* internal/querynodev2/segments/segment.go // (*LocalSegment).innerLoadIndex
* internal/querynodev2/segments/load_index_info.go // (*LoadIndexInfo).loadIndex
* internal/core/src/segcore/load_index_c.cpp // AppendIndexV2
* [milvus-common] include/cachinglayer/Manager.h // CreateCacheSlot
* [milvus-common] include/cachinglayer/CacheSlot.h // Warmup
* [milvus-common] include/cachinglayer/CacheSlot.h // {PinCells,PinInternal}
* [milvus-common] include/cachinglayer/CacheSlot.h // RunLoad
* [milvus-common] include/cachinglayer/Translator.h // get_cells
    internal/core/src/segcore/storagev1translator/SealedIndexTranslator.cpp

#### Load memory vector index

* internal/core/src/index/VectorMemIndex.cpp // Load
* internal/core/src/storage/MemFileManagerImpl.cpp // LoadIndexToMemory
* internal/core/src/index/VectorMemIndex.cpp // AssembleIndexDatas
    > Unpacks index_data_codecs, which is a map from file name to index codec, to binary_set, which is a map from index key to serialized index data, i.e. flattens file name.
    >
    > This step requires double the memory since both the input index_data_codecs and the output binary_set are present in memory.
* internal/core/src/index/VectorMemIndex.cpp // LoadWithoutAssemble
* [knowhere] src/index/index.cc // Deserialize
* [knowhere] src/index/{hnsw/faiss_hnsw.h} // Deserialize
    > FAISS read header, vector, index.
    >
    > Requires double the memory since both the BinarySet and the constructed faiss::Index coexist.

#### Load disk vector index

TODO

#### Load scalar index

TODO

### Load text index

TODO

* internal/querynodev2/segments/segment.go // (*LocalSegment).LoadTextIndex

### Load JSON index

TODO

* internal/querynodev2/segments/segment.go // (*LocalSegment).LoadJSONKeyIndex

### Load delta logs

TODO

* loader.loadDeltalogs
