---
layout: article
title: Ceph Deployment
key: ceph-deployment
tags: DistributedSystems Ceph Storage
---

<!-- more -->

# Ceph Deployment

## Basic Concepts

Ceph provides Ceph Object Storage, Ceph Block Device services to Cloud Platforms.

### Composition

A Ceph Storage Cluster requires at least one

* Ceph Monitor `ceph-mon`

    Maintains maps of cluster state.

* Ceph Manager `ceph-mgr`

    Keeps track of runtime metrics and current state of the Ceph cluster.

* Ceph Object Storage Daemon (OSD) `ceph-osd`

    Stores data, handles data replication, recovery, rebalancing, and provides
    some monitoring information to Ceph Monitors and Managers (by checking other
    Ceph OSD Daemons for heartbeat).

* Ceph Metadata Server (MDS) `ceph-mds`

    Stores metadata on behalf of Ceph File System (i.e., Ceph Block Devices and
    Ceph Object Storage do not use MDS).

<center><img src="https://docs.ceph.com/docs/master/_images/ditaa-b490c5d9d3bb6984503b59681d08337aff62e992.png"/></center>

### The CRUSH Algorithm

Ceph stores data as objects within logical storage pools. Using the CRUSH
algorithm, Ceph calculates which placement group should contain the object, and
further calculates which Ceph OSD Daemon should store the placement group.


## Installation and Deployment

### Service Node Setup

*Do this for all service nodes.*

1. User with password-less `sudo` privilege named `cephuser`

    ```console
    # useradd -d /home/ceph -m cephuser     # create user if not exists
    # passwd cephuser                       # enter password
    # echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph  # password-less sudo
    # chmod 0440 /etc/sudoers.d/ceph
    # sed -i "s/Defaults requiretty/#Defaults requiretty"g /etc/sudoers     # tty-less sudo
    ```

    > You may **NOT** name the user `ceph`, for Ceph software may use this name.

2. Check hostname

    Server nodes' hostnames must be identical to their hostnames on admin node.

3. Install NTP


### Admin Node Setup

1. Ensure connectivity with password-less SSH capability
    1. Add nodes to hosts for mnemonics purposes

        > `ceph-deploy` actually requires nodes to be specified by hostnames,
        > making this step necessary.

        ```text
        192.169.xxx.xxx ceph-mon1
        192.168.xxx.xxx ceph-osd1
        ```

    2. Configure password-less SSH connections

        > Otherwise `ceph-deploy` would prompt for password repeatedly.

        1. Initialize SSH keys

            ```console
            $ ssh-keygen
            ```

        2. Associate aliases to SSH connections

            ```text
            # ~/.ssh/config
            Host ceph-mon1
                Hostname ceph-mon1      # the hostname in /etc/hosts
                User ceph               # Ceph user on service node
            ```

            ```console
            $ chmod 644 ~/.ssh/config
            ```

        3. Store public keys

            ```console
            $ ssh-keyscan ceph-mon1 ceph-osd1 ... >> ~/.ssh/known_hosts
            $ ssh-copy-id ceph-mon1     # enter passwords once and for all!
            $ ssh-ccpy-id ceph-osd1
            $ ...
            ```

2. Install `ceph-deploy`

    ```console
    # apt install ceph-deploy       # Debian / Ubuntu

    # yum install epel-release      # RHEL / CentOS
    # yum install ceph-deploy
    ```

### Deployment

> Switch to a dedicate working directory, say `mkdir cluster && cd cluster`,
> before you proceed.

1. Create storage cluster

    ```console
    $ ceph-deploy new ceph-osd1 ceph-osd2 ...
    ```

    > This step only ensures connectivity and generates a configuration file
    > on current admin node's working directory. Actual storage service is not
    > up yet.

2. Install software on all nodes

    ```console
    $ ceph-deploy install [--release {release}] ceph-mon1 ceph-osd1 ...
    ```

    > Packages in official CentOS repo is outdated, and may try to download from
    > `http://ceph.com/xxxx`, not the updated `https://download.ceph.com/xxxx`.
    > To correct this, you may
    > 1. <span></span>
    >     ```console
    >     $ export CEPH_DEPLOY_REPO_URL=http://mirrors.163.com/ceph/{dist}-{release}
    >     $ export CEPH_DEPLOY_GPG_URL=http://mirrors.163.com/ceph/keys/release.asc
    >     ```
    > 2. edit URLs in
    >     `/usr/lib/python2.7/site-packages/ceph_deploy/hosts/centos/install.py`.

    > To **truely** uninstall Ceph,
    >
    > ```console
    > $ ceph-deploy purge node1 node2 ...
    > $ ceph-deploy purgedata node1 node2 ...
    > $ ceph-deploy forgetkeys
    > # yum remove `yum list installed | grep ceph | awk '{print $1}'`
    > # rm -rf /var/run/ceph
    > # vgremove ceph-xxxxx
    > ```

> If you have more than one network interface, you must have the following option
> configured and pushed to service nodes.
>
> ```toml
> [global]
> public_network=xxx.xxx.xxx.xxx/xx
> ```
>
> Push config file and restart Ceph service to make new configs in effect.
>
> ```console
> $ ceph-deploy --overwrite-conf config push node1 node2 ...
> $ ssh node1 sudo systemctl restart ceph.target
> ```

> Deploying Ceph on a one-node cluster can cause deadlock!
>
> Before you continue to create monitors, you must have the following options
> configured and pushed to service nodes.
>
> ```toml
> [global]
> osd_crush_chooseleaf_type = 0     # let CRUSH navigate to OSDs instead of hosts
> osd_pool_default_size = 1
> osd_pool_default_min_size = 1     # allow PGs to provide service at 'undesired' state
> ```
>
> See [here](https://docs.ceph.com/docs/master/rados/troubleshooting/troubleshooting-pg/#one-node-cluster)
> and [here](https://tracker.ceph.com/projects/ceph/wiki/How_Can_I_Give_Ceph_a_Try)
> before you proceed.

3. Create monitors

    ```console
    $ ceph-deploy mon create ceph-mon1
    $ ceph-deploy gatherkeys ceph-mon1
    ```

    > On CentOS, if encountered error throwing this message
    >
    > ```text
    > [ceph-deploy.mon][ERROR] Failed to execute command: /usr/sbin/service ceph -c /etc/ceph/ceph.conf start mon.{node}
    > ```
    >
    > It may due to `ceph-deploy` being outdated (for it's invoking `service`
    > instead of the more mordern, feature-rich `systemctl`), or lack of LSB
    > (Linux Standard Base) support.
    >
    > Fix it by installing the latest `ceph-deploy` scripts and the `redhat-lsb`
    > package.

    > If your monitor starts but reported to be stuck at `probing` state, there
    > is a good chance that IP addresses for monitors in `monmap` are broken.
    > Please ensure that
    >
    > 1. The IPs are correct
    > 2. No mixed use of IPv4 and IPv6 (e.g. using IPv4 for `public_network`
    >     with `ms_bind_ipv6 = true`)

    > If your monitor starts but reported to be stuck at `electing` state, you
    > may have a *clock drift* situation. You may find the following configuration
    > helpful.
    >
    > ```toml
    > [global]
    > mon_force_quorum_join = true      # prefered if debugging
    > mon_clock_drift_allowed = 5.0     # in seconds
    > ```

4. Register admin node of the cluster

    ```console
    $ ceph-deploy admin ceph-admin
    ```

    Now you may use `ceph` CLI tool without having to specify the monitor address
    and `ceph.client.admin.keyring` before you execute a command.

5. Deploy a manager daemon

    ```console
    $ ceph-deploy mgr create ceph-mgr1
    ```

6. Add OSDs

    ```console
    $ ceph-deploy osd create ceph-osd1:vdb ceph-osd2:vdb ceph-osd3:vdb
    ```

7. Check cluster health

    ```console
    $ ssh ceph-node sudo ceph health
    $ ssh ceph-node sudo ceph -s
    ```

8. \[Optional] Testrun: create / delete object

    ```console
    $ echo 'something' > testfile
    # ceph osd pool create mytest 30    # 30 PGs, too few will cause RADOS to deadlock
    # rados --pool=mytest put test-obj1 ./testfile
    # rados -p mytest ls
    # rados -p mytest rm test-obj1
    ```

## Cache Tiering

* [Cache Tiering - Ceph Documentation](https://docs.ceph.com/docs/master/rados/operations/cache-tiering/)

<center><img src="https://docs.ceph.com/docs/master/_images/ditaa-2982c5ed3031cac4f9e40545139e51fdb0b33897.png"/></center>

The Ceph objecter handles where to place the objects and the tiering agent
determines when to flush objects from the cache to the backing storage tier.

The cache tier and the backing storage tier are completely transparent to Ceph
clients.

### `cache-mode`

* `writeback`

    Flushing is performed asynchronously.

* `readproxy`

    Use any objects that already exists in the cache tier, but does **not**
    migrate data on cache miss.

    Useful for transitioning from `writeback` mode to disable cache.

* `readonly`

    Promotes objects to cache on read operations only, write operations are
    forwarded to base tier.

    Updates on base tier are **not** synced to cache tier.

* `none`

### Creating a Cache Tier

Settings up a cache pool follows the same procedure as the standard storage
scenario, but with this difference: the drives for the cache tier are typically
high performance drives that reside in their own servers and have their own
CRUSH rule. When setting up such a rule, it should take account of the hosts
that have the high performance drives while omitting the hosts that don't.
See [Manually Editing a CRUSH Map](https://docs.ceph.com/docs/master/rados/operations/crush-map-edits/#placing-different-pools-on-different-osds)
for details.

> You may manually assign device class to devices.
>
> ```console
> # ceph osd crush rm-device-class osd.2 osd.3
> # ceph osd crush set-device-class ssd osd.2 osd.3
> ```

```console
# ceph osd tier add base-tier-pool cache-tier-pool
# ceph osd tier cache-mode cache-tier-pool writeback
# ceph osd tier set-overlay base-tier-pool cache-tier-pool
# ceph osd pool set cache-tier-pool target_max_bytes {size}
```

> Ceph cannot determin size of cache pool automatically, thus configuration on
> absolute size is required, otherwise flush / evict will not work.

> All client requests **will be blocked** when `target_max_bytes` or
> `target_max_objects` reached, and flushing / evicting will start.

Changes to base tier will now be handled by cache tier.

<center><img src="https://raw.githubusercontent.com/smdsbz/smdsbz.github.io/master/assets/images/2019-10-15-ceph-notes/cache-tier-testrun.png"/></center>








---------------------------------------------

## References

* [Ceph Documentation](https://docs.ceph.com/docs/master/)
* [How to Build a Ceph Distributed Storage Cluster on CentOS 7](https://www.howtoforge.com/tutorial/how-to-build-a-ceph-cluster-on-centos-7/)
