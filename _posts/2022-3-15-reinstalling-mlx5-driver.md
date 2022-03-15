---
layout: article
title: Reinstalling Driver for Mellanox Connect-X 5
key: troubleshooting-mlx5
tags: RDMA NVidia Mellanox
---

Network interface `ibxxx` disapeared after reboot? Try reinstalling driver!

<!-- more -->

1. Check system distribution

    ```console
    ~# cat /etc/issue 
    Ubuntu 21.10 \n \l           
    ```

2. Download driver for your system from
    [MLNX_OFED Download Center](https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/)
3. Unzip and install with

    ```console
    ~# tar -zxvf MLNX_...tgz && cd MLNX_...
    ~# ./mlxnofedinstall
    ```

4. Wait
5. Restart IB service

    ```console
    ~# /etc/init.d/openibd restart
    ```

4. Verify

    ```console
    ~# ip addr
    ...
    7: ibp175s0: <BROADCAST,...
        link/infiniband 00:0...
    ```

__Similar issue__

* [SOLVED: No infiniband device ib0 after upgrade to Centos 7.3](https://forums.centos.org/viewtopic.php?t=61254)
