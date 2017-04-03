---
layout:     post
title:      "给qcow2格式的磁盘的kvm扩容"
subtitle:   "磁盘扩容"
date:       "2017-04-03"
author:     "xiechengsheng"
header-img: "img/post-bg-4.jpg"
catalog: true
tags:
    - tools
---

在KVM客户机运行一段时间之后，由于默认安装的磁盘容量为20G，虚拟机磁盘的可用空间会严重不足，导致整个虚拟机运行不正常，需要将物理机的磁盘空间多分配点供虚拟机使用；

# 使用指令给qcow2格式的磁盘的kvm虚拟机扩容
1. 定制各个虚拟机的配置的xml文件默认在宿主机的这个目录：`/etc/libvirt/qemu/`
虚拟机的镜像存储默认在宿主机的这个目录：`/var/lib/libvirt/images/`

2. 查看虚拟机磁盘信息：
```sh
# qemu-img info /var/lib/libvirt/images/vm6.qcow2
image: /var/lib/libvirt/images/vm6.qcow2
file format: qcow2
virtual size: 40G (42949672960 bytes)
disk size: 40G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
```

3. 使用直接添加虚拟机原始镜像的方法给虚拟机磁盘扩容；

4. 在宿主机上的操作：
    - 强制关闭虚拟机，选择shutdown虚拟机可能会出问题：`virsh destroy vm6`
    - 给虚拟机磁盘添加10G空间：`qemu-img resize vm6.qcow2 +10G`
    - 开机：
        ```sh
        virsh start vm6
        virsh list
        ```

5. 在客户机上的操作：
    - 查看磁盘分区，可以发现已经增加了10G：`fdisk -l`
    - 查看虚拟机磁盘挂载的空间，发现没有增加：`df -h`
    - 使用fdisk将新增加的磁盘空间使用上：
    ```sh
        # fdisk /dev/vda
        Welcome to fdisk (util-linux 2.23.2).

        Changes will remain in memory only, until you decide to write them.
        Be careful before using the write command.

        Command (m for help): p

        Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
        Units = sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disk label type: dos
        Disk identifier: 0x000aebb5

        Device Boot      Start         End      Blocks   Id  System
        /dev/vda1   *        2048     1026047      512000   83  Linux
        /dev/vda2         1026048    62914559    30944256   8e  Linux LVM

        Command (m for help): n
        Partition type:
        p   primary (2 primary, 0 extended, 2 free)
        e   extended
        Select (default p): p
        Partition number (3,4, default 3): 3
        First sector (62914560-83886079, default 62914560):
        Using default value 62914560
        Last sector, +sectors or +size{K,M,G} (62914560-83886079, default 83886079):
        Using default value 83886079
        Partition 3 of type Linux and of size 10 GiB is set

        Command (m for help): p

        Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
        Units = sectors of 1 * 512 = 512 bytes
        Sector size (logical/physical): 512 bytes / 512 bytes
        I/O size (minimum/optimal): 512 bytes / 512 bytes
        Disk label type: dos
        Disk identifier: 0x000aebb5

        Device Boot      Start         End      Blocks   Id  System
        /dev/vda1   *        2048     1026047      512000   83  Linux
        /dev/vda2         1026048    62914559    30944256   8e  Linux LVM
        /dev/vda3        62914560    83886079    10485760   83  Linux

        Command (m for help): w
        The partition table has been altered!

        Calling ioctl() to re-read partition table.

        WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
        The kernel still uses the old table. The new table will be used at
        the next reboot or after you run partprobe(8) or kpartx(8)
        Syncing disks.
    ```
    - 重启客户机生效：`reboot`
    - 查看系统PV：
        ```sh
        pvdisplay
        pvs
        ```
    - 把/dev/vda3 加入到lvm里面去：`pvcreate /dev/vda3`
    - 查看系统的VG：`vgdisplay`，其输出应该为，后面需要用到卷组（Volume Group）：
        ```sh
        --- Volume group ---
        VG Name               centos
        ```
    - 将物理卷加入卷组：`vgextend centos /dev/vda3`
    - 核对查看卷组的剩余空闲空间：`vgs`
    - 查看逻辑卷：lvs，输出信息应该为：
        ```sh
        LV   VG         ...
        root centos   ...
        ```
    - 将剩余的空闲卷加入到root文件系统下：`lvextend -l +100%FREE /dev/centos/root`
    - 检查客户机逻辑卷：`e2fsck -f /dev/centos/root`
    - 重新刷新root分区大小：
        ```sh
        resize2fs /dev/centos/root     （可能会报错，使用下面的指令）
        xfs_growfs /dev/centos/root
        ```
    - 重新查看客户机磁盘大小，此时发现已经正确扩充磁盘容量：`df -h`



# 参考
[KVM虚拟化基本管理](http://www.2cto.com/os/201511/449832.html)
[linux下 lvm 磁盘扩容](http://www.cnblogs.com/einyboy/archive/2012/05/31/2528661.html)
[LVM XFS增加硬盘分区容量(resize2fs: Bad magic number in super-block while)](http://www.cnblogs.com/archoncap/p/5442208.html)
