---
layout: article
title: Ceph and Paxos
key: ceph-and-paxos
tags: DistributedSystems Ceph Paxos Consensus
---

Lines from ["Ceph Monitors and Paxos, a chat with Joao"](1).

<!-- more -->

# Ceph and Paxos

## The "Leasing" Mechanism

> **Loic:** the last point is the leasing mechanism : 3- A ‘leasing’ mechanism
> is built-in, allowing nodes to determine when it is safe to “read” their copy
> of the last committed value.
>
> **Joao:** you have a leader, proposes a given paxos version. The other
> monitors commit that version. From that point on it becomes available to be
> readable by any client from any monitor. However, that version has a time to
> live. The reading capabilities on a given monitor has a time to live. Let say
> you have three monitors and you have clients connected to all of them. If one
> of the monitors loses touch with the other monitors. It is bound to drop out
> of the quorum ( and it is unable to receive new versions ). The time to live
> is assigned to a given paxos version : the last_committed version. There are
> not multiple leases because it’s all it requires. That lease will have to be
> refreshed by the leader : if it expires the monitor will assume it lost
> connection with the rest of the cluster, including the leader.
>
> **Loic:** and it will cease to serve the clients requests ?
>
> **Joao:** yes, and bootstrap a new election.
>
> **Loic:** the benefit is that when a client is connected to a monitor that
> lost touch with the quorum, it will keep getting data, but not for too long.
>
> **Joao:** and I believe we rely on the lease time out to trigger the election.
> *It can also mean that the leader died, which also requires a new election.*
> You can imagine now why latency and clock synchronization is so critical in
> the monitors. *If you have a clock skew it means you will expire either
> earlier or later. Regardless it will create chaos and randomness and monitors
> will start calling for elections constantly.*

1. The *time to live* is configured among all Paxos learners. On expiration,
    the previous version of proposition is no longer valid.

2. It seems Ceph relies on an authouritative clock for checking expiration, if
    clocks are un-synchronized, the `last_committed` version will not expire on
    all learners **simultaneously**.

    Internally, Ceph uses `ceph::real_clock::now()` for lease time calculation,
    which effectively calls `gettimeofday()` (for Linux; if on a Mac it's
    `mach_absolute_time()`).

3. The "leasing" mechanism is also used as heartbeat: if a node does not respond
    to a new round of election, it is considered down.



------------------------------------------

## Reference

* [Ceph Monitors and Paxos, a chat with Joao](1)


[1]: https://ceph.io/geen-categorie/monitors-and-paxos-a-chat-with-joao/
