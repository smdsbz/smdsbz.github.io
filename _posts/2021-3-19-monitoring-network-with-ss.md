---
layout: article
title: Monitoring Network With ss
key: monitoring-network-with-ss
tags: Linux Network Monitor StatisticsCollection
---

<!-- more -->

Monitoring Network With `ss`
===============================

There are multiple utilities comes with most Linux distributions for TCP digging.

- `netstat`: obsolete
- `nstat`: dumps kernel snmp counters
- `ss`: a comprehensive tool, used to replace `netstat`
- `ip tcp_metrics`: if you want more details about TCP connection parameters


Use Case
--------

__Get packets going in / out a certain address__

```console
$ ss -a -i src 127.0.0.1:3260
Netid  Recv-Q Send-Q                                         Local Address:Port                                                          Peer Address:Port                
tcp    0      0                                                  127.0.0.1:iscsi-target                                                     127.0.0.1:53144                
         cubic wscale:2,7 rto:205 rtt:4.561/9.094 ato:40 mss:65483 rcvmss:536 advmss:65483 cwnd:10 ssthresh:7 bytes_acked:349292 bytes_received:4068 segs_out:203 segs_in:174 send 1148.6Mbps lastsnd:998 lastrcv:998 lastack:958 pacing_rate 2297.0Mbps rcv_space:43690
```

- `-a`: list all sockets
- `-i`: show info about TCP

- `bytes_acked`: bytes acked _by peer_, a.k.a. bytes sent successfully
- `bytes_reveived`: bytes received by self

> RFC4898 named these, here's some comments found in Linux [source code](https://elixir.bootlin.com/linux/latest/source/include/linux/tcp.h#L157) :sweat_smile:
>
> ```c
> struct tcp_sock {
> ...
> /*
>  *	RFC793 variables by their proper names. This means you can
>  *	read the code and the spec side by side (and laugh ...)
>  *	See RFC793 and RFC1122. The RFC writes these in capitals.
>  */
> 	u64	bytes_received;	/* RFC4898 tcpEStatsAppHCThruOctetsReceived
> 				 * sum(delta(rcv_nxt)), or how many bytes
> 				 * were acked.
> 				 */
> ...
> 	u64	bytes_sent;	/* RFC4898 tcpEStatsPerfHCDataOctetsOut
> 				 * total number of data bytes sent.
> 				 */
> 	u64	bytes_acked;	/* RFC4898 tcpEStatsAppHCThruOctetsAcked
> 				 * sum(delta(snd_una)), or how many bytes
> 				 * were acked.
> 				 */
> ...
> }
> ```

For more options, see [`man ss`](https://man.archlinux.org/man/ss.8.en).
