---
layout: article
title: Uniquely Identifying Mounted iSCSI Device
key: uniquely-identifying-mounted-iscsi-device
tags: Linux iSCSI VirtualFileSystem
---

<!-- more -->

`iscsiadm` documentation is trash!

Linux sometimes truncates LUN name when they are long enough (historical design
flaw, and they usually are), and you may fail to uniquely locate your iSCSI
device with `lsscsi` if they unfortunately share a prefix.

Use the most verbose `iscsiadm` output to get the IQN and its device name of an
active session.

```console
# iscsiadm --mode session -P3   # print session info with detail of level 3
iSCSI Transport Class version 2.0-870
version 6.2.0.874-17
Target: iqn.2020-07.com.hikstorage.iscsi:torch1512 (non-flash)
        Current Portal: 192.168.3.146:3260,1
        Persistent Portal: 192.168.3.146:3260,1
          **********
          Interface:
          **********
          Iface Name: default
          Iface Transport: tcp
          Iface Initiatorname: iqn.1994-05.com.redhat:863513e73063
          Iface IPaddress: 192.168.3.146
          Iface HWaddress: <empty>
          Iface Netdev: <empty>
          SID: 408
          iSCSI Connection State: LOGGED IN
          iSCSI Session State: LOGGED_IN
          Internal iscsid Session State: NO CHANGE
          *********
          Timeouts:
          *********
          Recovery Timeout: 120
          Target Reset Timeout: 30
          LUN Reset Timeout: 30
          Abort Timeout: 15
          *****
          CHAP:
          *****
          username: <empty>
          password: ********
          username_in: <empty>
          password_in: ********
          ************************
          Negotiated iSCSI params:
          ************************
          HeaderDigest: None
          DataDigest: None
          MaxRecvDataSegmentLength: 262144
          MaxXmitDataSegmentLength: 262144
          FirstBurstLength: 65536
          MaxBurstLength: 262144
          ImmediateData: Yes
          InitialR2T: Yes
          MaxOutstandingR2T: 1
          ************************
          Attached SCSI devices:
          ************************
          Host Number: 410      State: running
          scsi410 Channel 00 Id 0 Lun: 0
                Attached scsi disk sdb    State: running
```

On the last line, you have the actual name of the mounted device.
