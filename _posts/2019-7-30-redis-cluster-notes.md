---
layout: article
title: Redis Cluster Notes
key: redis-cluster-note
tags: DistributedSystems Redis RedisCluster
---

> Based on unstable branch at 7/25/2019.

<!-- more -->

Redis Cluster
=============

Redis Cluster Main Components
-----------------------------

__Key Distribution Model__

$$
hash\_slot = \text{CRC16}(key) \ \mathbf{mod} \ 2^{14}
$$

__Keys Hash Tags__

Redis Cluster implements a concept called hash tags that can be used in order to
force certain keys to be stored in the same node.

Only hash contents between the first occurances of `{` and `}`, otherwise
hash the whole key.

__Cluster Nodes Attribute__

Every node has a unique ID (usually obtained from `/dev/urandom` on
initialization) and is globally consistent.

It's posibble for a given node to change its IP address without changing
node ID, the cluster is able to detect such change using the gossip protocal
on the cluster bus.

__Cluster Topology__

Redis Cluster is a __full mesh__ where every node is connected to every other
node using a TCP connection.

> In a cluster of $$N$$ nodes, every node has $$N - 1$$ outgoing TCP connections
> and $$N - 1$$ incomming connections.

Redirection and Resharding
--------------------------

__`MOVED` Redirection__

A Redis client is free to send queries to every node in the cluster. If the
query is acceptable (i.e. only a single key is mentioned in the query, or all
keys mentioned points to the same hash slot), the node will lookup which node is
responsible for the hash slot.

If the hash slot is served by the node, the query is processed, otherwise the
node will check its __internal hash-slot-to-node map__, and reply the client
with a `MOVED` error.

    -MOVED <hash_slot> <ip>:<port>

The client then reissue query to the returned node address.

Note that the other node may _again_ return a `MOVED` if the cluster is
reconfigured just before the reissuing.

It's recommended for the client to try to remember the hash-slot-to-node
mapping, or just `CLUSTER NODES` or `CLUSTER SLOTS` to grep the full mapping.

__Cluster Live Reconfiguration__

Adding or removing a node is abstracted into the same operation: moving a hash
slot from one node to another, which means the same basic mechanism can be used
for rebalancing.

- To add a new node to the cluster

    1. An empty node is added to the cluster

        ```console
        $ redis-cli --cluster add-node \
        <address-of-new-master> \
        <address-of-node-with-conf>
        ```

    2. Some set of hash slots are moved from existing node to the new node

        _Do this manually by calling `reshard`_

        ```console
        $ redis-cli --cluster reshard \
        <address-of-new-master>
        ```

- To remove a node from the cluster

    1. Hash slots assigned to that node are moved to other existing nodes
    2. Remove the node

    > A master node must be empty to be removed. Which means you must reshard
    > hash slots away before you can successfully remove the node.
    >
    > Alternatively, maually failover the master over its slave, a chosen slave
    > will turn into master, while the fileover-ed master will then become a
    > slave, then you can safely remove the node.
    >
    > However, if your intention were to reduce the total number of masters in
    > the cluster, a manual resharding is still needed.

- To rebalance the cluster

    1. A given set of hash slots are moved between nodes

A hash slot can be set in one of two special states

- `MIGRATING`

    Will accept query if the key still exists, otherwise forwarded with `-ASK`
    redirection.

- `IMPORTING`

    Will accept query if the key exists and the query is preceded by `ASKING`
    command. If the `ASKING` is not given, the query is redirected to the real
    hash slot owner via `-MOVED` redirection error.

__After a `reshard` Is Issued...__

Call stack (`src/redis-cli.c`)

- `clusterManagerCommandReshard`
  
    - `clusterManagerCheckCluster`
        - `clusterManagerIsConfigConsistent`
        - for each slot, check
            - `n->migrating`
            - `n->importing`
            - `open_slots`
            - `clusterManagerFixMultipleSlotOwners`
    - get target node to `MIGRATE` to by calling `clusterNodeForResharding`
    - get list of source slots (and the nodes they're on) by calling
    `clusterManagerComputeReshardTable`
    
        > __Sidenotes on implementation__
        >
        > The migration is issued evenly accross slots, with only bit more focus
        > on nodes with more slots.
        >
        > ```c
        > ...
        > qsort(sorted, src_count, sizeof(clusterManagerNode *),
        >    clusterManagerSlotCountCompareDesc);
        > for (i = 0; i < src_count; i++) {
        >  ...
        >  float n = ((float) numslots / tot_slots * node->slots_count);
        >  if (i == 0) n = ceil(n);
        >  else n = floor(n);
        >  ...
        > }
        > ```
    
    - for each `clusterManagerResharTableItem` in the reshard table, run `clusterManagerMoveSlot`
        - for target, issue `CLUSTER SETSLOT <slot> IMPORTING <source-id>`
        - for source, issue `CLUSTER SETSLOT <slot> MIGRATING <target-id>`
        - `clusterManagerMigrateKeysInSlot`
            - (loop forever)
                - for source, issue `CLUSTER GETKEYSINSLOT <slot> <pipeline>`
                - if reply is not empty
                    - `clusterManagerMigrateKeysInReply`
                        - for source, issue  
                            `MIGRATE <target-addr> "" 0 <timeout> [AUTH ...] KEYS ...`
                - else, break and return
        - for each node in the cluster, notify the change by issuing  
            `CLUSTER SETSLOT <slot> NODE <target-id>`

Configuration Handling, Propagation and Failovers
-------------------------------------------------

__Slave election and promotion__

- `currentEpoch`: similar to "Raft" algorithm _term_, set to 0 at node creation
    (both masters and slaves), every time a packet received from another node,
    `currentEpoch` gets updated to the greater one.

    > Currently only used during slave promotion.

- `configEpoch`: set to 0 _in masters_ when a new node is created, i.e.
    `configEpoch` tells how long the last stable configuration lasted.

Slave election and promotion is handled by slave nodes, with help of master
nodes that vote for the slave to promote:

1. Votes are requested by the slave by broadcasting `FAILOVER_AUTH_REQUEST` to
    every master in the cluster, and wait at most `2 * NODE_TIMEOUT` (but at
    least 2 sec).
2. Master votes for a slave by replying an `AUTH_REQUEST`, whose `currentEpoch`
    larger than the recorded `lastVoteEpoch`, with `AUTH_ACK` and stops voting
    for other slaves, or more precisely, other slaves of the same failed master,
    for a period of `2 * NODE_TIMEOUT`.
3. A slave discards any `AUTH_ACK` with smaller `configEpoch`.
4. Once a slave wins a majority of `AUTH_ACK`, it wins the election.

__Replica migration algorithm__

Replica migration is the process of automatically reconfiguration of slaves in
order to migrate to a master that has no longer coverage (i.e. no working
slaves). The algorithm guarantees that eventually (one the cluster configuration
is stable) every master will be backed by at least one slave.

The algorithm is triggered in every slave that detects there is at leaset master
without working slaves. The subset of _acting slaves_ is defined to be the among
the slaves of the master with most slaves connected to, and with the lowest ID.
Usually the subset only contain one acting slave, unless the configuration is
not sync-ed across the cluster (but such race condition is however harmless).

Partitioning: how to split data among multiple Redis instances
--------------------------------------------------------------

__Different implementations of partitioning__

- Client side partitioning

    The client directly select the right node where to read and write.

    Many Redis clients implement client side partitioning.

- Proxy assisted partitioning

    Clients send requests to proxy speaking Redis protocal, the proxy will make
    sure to forward requests to the right Redis instance.

    The Redis and Memcached proxy Twemproxy implements proxy assisted partitioning.

- Query routing

    Query sent to a random instance, the instance forwards the query to the
    right node.

    Redis Cluster implements an hybrid form of query routing, with the help of
    client (client handles redirection).

-----------------------------------------------------------------------

Reference
=========

- [Redis Cluster tutorial](https://redis.io/topics/cluster-tutorial)
- [Redis Cluster Specification](https://redis.io/topics/cluster-spec)
- [Partitioning: how to split data among multiple Redis instances](https://redis.io/topics/partitioning)