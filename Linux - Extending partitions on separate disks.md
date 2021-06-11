# Extending partitions on separate disks
>Found at https://www.thegeekstuff.com/2010/08/how-to-create-lvm/

1. Log in as root
1. Add the new disk/partition to the VM (Adding a drive)
1. Create the physical volumes
    ```
    pvcreate /dev/sdb1
    ```
1. View the physical volumes on the VM
    ```
    pvscan

    PV /dev/sda2   VG centos          lvm2 [<31.00 GiB / 0    free]
    PV /dev/sda3   VG centos          lvm2 [<68.00 GiB / 0    free]
    PV /dev/sdb1   VG centos          lvm2 [<100.00 GiB / 0    free]
    Total: 3 [<198.99 GiB] / in use: 3 [<198.99 GiB] / in no VG: 0 [0   ]
    ```
    1. View the list of physical volumes with attributes
        ```
        pvdisplay
        
        --- Physical volume ---
        PV Name               /dev/sda2
        VG Name               centos
        PV Size               <31.00 GiB / not usable 3.00 MiB
        Allocatable           yes (but full)
        PE Size               4.00 MiB
        Total PE              7935
        Free PE               0
        Allocated PE          7935
        PV UUID               sE621r-IEdN-ytNH-1rBp-IWBK-L95B-1ZYRZe
        
        --- Physical volume ---
        PV Name               /dev/sda3
        VG Name               centos
        PV Size               68.00 GiB / not usable 4.00 MiB
        Allocatable           yes (but full)
        PE Size               4.00 MiB
        Total PE              17407
        Free PE               0
        Allocated PE          17407
        PV UUID               yFZ7xY-tTqz-SqTs-22YB-z9h8-35eH-PTxZEN
        
        --- Physical volume ---
        PV Name               /dev/sdb1
        VG Name               centos
        PV Size               <100.00 GiB / not usable 3.00 MiB
        Allocatable           yes (but full)
        PE Size               4.00 MiB
        Total PE              25599
        Free PE               0
        Allocated PE          25599
        PV UUID               M5usB0-Ri0N-ijZy-48Ty-iiCr-2IVA-iZOU5L
        ```
1. Follow instructions from `Extending partitions on same disk` starting at instruction **14. Add the physical volume to the existing volume group using the vgextend command**.

## Commands I ran for the computermachine VM when adding a third disk (/dev/sdc) to increase the size of the root partition to 300GB
```
[root@computermachine ~]# fdisk -l

Disk /dev/sdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xeee3514d

Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   209715199   104856576   83  Linux

Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b23e8

Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    67108863    32504832   8e  Linux LVM
/dev/sda3        67108864   209715199    71303168   83  Linux

Disk /dev/mapper/centos-root: 210.2 GB, 210226905088 bytes, 410599424 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 3435 MB, 3435134976 bytes, 6709248 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
```
[root@computermachine ~]# fdisk /dev/sdc
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x48a369c6.

Command (m for help): n
Partition type:
p   primary (0 primary, 0 extended, 4 free)
e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-209715199, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-209715199, default 209715199):
Using default value 209715199
Partition 1 of type Linux and of size 100 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
```
[root@computermachine ~]# fdisk -l

Disk /dev/sdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xeee3514d

Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   209715199   104856576   83  Linux

Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b23e8

Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    67108863    32504832   8e  Linux LVM
/dev/sda3        67108864   209715199    71303168   83  Linux

Disk /dev/mapper/centos-root: 210.2 GB, 210226905088 bytes, 410599424 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 3435 MB, 3435134976 bytes, 6709248 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x48a369c6

Device Boot      Start         End      Blocks   Id  System
/dev/sdc1            2048   209715199   104856576   83  Linux
```
```
[root@computermachine ~]# pvscan
PV /dev/sda2   VG centos          lvm2 [<31.00 GiB / 0    free]
PV /dev/sda3   VG centos          lvm2 [<68.00 GiB / 0    free]
PV /dev/sdb1   VG centos          lvm2 [<100.00 GiB / 0    free]
Total: 3 [<198.99 GiB] / in use: 3 [<198.99 GiB] / in no VG: 0 [0   ]
```
```
[root@computermachine ~]# pvcreate /dev/sdc1
Physical volume "/dev/sdc1" successfully created.
```
```
[root@computermachine ~]# pvscan
PV /dev/sda2   VG centos          lvm2 [<31.00 GiB / 0    free]
PV /dev/sda3   VG centos          lvm2 [<68.00 GiB / 0    free]
PV /dev/sdb1   VG centos          lvm2 [<100.00 GiB / 0    free]
PV /dev/sdc1                      lvm2 [<100.00 GiB]
Total: 4 [<298.99 GiB] / in use: 3 [<198.99 GiB] / in no VG: 1 [<100.00 GiB]
```
```
[root@computermachine ~]# vgextend centos /dev/sdc1
Volume group "centos" successfully extended
```
```
[root@computermachine ~]# vgdisplay --units m
--- Volume group ---
VG Name               centos
System ID
Format                lvm2
Metadata Areas        4
Metadata Sequence No  10
VG Access             read/write
VG Status             resizable
MAX LV                0
Cur LV                2
Open LV               2
Max PV                0
Cur PV                4
Act PV                4
VG Size               306160.00 MiB
PE Size               4.00 MiB
Total PE              76540
Alloc PE / Size       50941 / 203764.00 MiB
Free  PE / Size       25599 / 102396.00 MiB
VG UUID               tWdVNH-54Ow-Ug3G-t8fH-zNNs-tegk-0BKJCP
```
```
[root@computermachine ~]#  lvextend -L+102396M /dev/centos/root
Size of logical volume centos/root changed from <195.79 GiB (50122 extents) to <295.79 GiB (75721 extents).
Logical volume centos/root successfully resized.
```
```
[root@computermachine ~]# vgdisplay --units m
--- Volume group ---
VG Name               centos
System ID
Format                lvm2
Metadata Areas        4
Metadata Sequence No  11
VG Access             read/write
VG Status             resizable
MAX LV                0
Cur LV                2
Open LV               2
Max PV                0
Cur PV                4
Act PV                4
VG Size               306160.00 MiB
PE Size               4.00 MiB
Total PE              76540
Alloc PE / Size       76540 / 306160.00 MiB
Free  PE / Size       0 / 0 MiB
VG UUID               tWdVNH-54Ow-Ug3G-t8fH-zNNs-tegk-0BKJCP
```
```
[root@computermachine ~]# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=29, agsize=1821440 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=51324928, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=3557, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 51324928 to 77538304
```
```
[root@computermachine ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  296G   94G  202G  32% /
devtmpfs                 2.9G     0  2.9G   0% /dev
tmpfs                    3.0G     0  3.0G   0% /dev/shm
tmpfs                    3.0G   34M  2.9G   2% /run
tmpfs                    3.0G     0  3.0G   0% /sys/fs/cgroup
/dev/sda1               1014M  249M  766M  25% /boot
tmpfs                    597M   12K  597M   1% /run/user/42
tmpfs                    597M     0  597M   0% /run/user/0
```