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

1. User with password-less `sudo` privilege named `ceph`

    ```console
    # useradd -d /home/ceph -m ceph     # create user if not exists
    # passwd ceph                       # enter password
    # echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph  # password-less sudo
    # chmod 0440 /etc/sudoers.d/ceph
    # sed -i "s/Defaults requiretty/#Defaults requiretty"g /etc/sudoers     # tty-less sudo
    ```

2. [Required for CentOS] Enable source of wanted Ceph release

    > Otherwise will complain package `ceph`, `ceph-radowsgw` cannot be found.

    ```console
    # yum install centos-release-ceph-{release}
    ```

    > `yum search ceph` for all available Ceph releases.

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

    > The default release is hard-coded into
    > `/usr/lib/python2.7/site-packages/ceph_deploy/install.py`.

    > Packages in official CentOS repo is outdated, and may try to download from
    > `http://ceph.com/xxxx`, not the updated `https://download.ceph.com/xxxx`.
    > To correct this, you may edit URLs in
    > `/usr/lib/python2.7/site-packages/ceph_deploy/hosts/centos/install.py`.

    > To **truely** uninstall Ceph,
    >
    > 1. Uninstall Ceph software
    >
    >     ```console
    >     $ ceph-deploy uninstall ceph-mon1 ceph-osd1 ...
    >     ```
    >
    > 2. Remove repository
    >
    >     ```console
    >     # yum remove centos-release-ceph-{release}
    >     ```
    >
    > 3. Manuall uninstall all python dependencies
    >
    >     ```console
    >     # yum remove `yum list installed | grep ceph-{release} | awk '{print $1}'`
    >     ```

3. Create monitors








---------------------------------------------

## References

* [Ceph Documentation](https://docs.ceph.com/docs/master/)
* [How to Build a Ceph Distributed Storage Cluster on CentOS 7](https://www.howtoforge.com/tutorial/how-to-build-a-ceph-cluster-on-centos-7/)
* [Using Ceph as Block Device on CentOS 7](https://www.howtoforge.com/tutorial/using-ceph-as-block-device-on-centos-7/)
