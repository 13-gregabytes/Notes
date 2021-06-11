# Extending partitions on same disk

>Found at https://www.techrepublic.com/blog/smb-technologist/extending-partitions-on-linux-vmware-virtual-machines/

1. Log in as root
1. At the command prompt type
    ```
    fdisk -l
    ```
1. The response should say something like
    ```
    Disk /dev/sda : xxGB
    ```
    EG:
    ```
    Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x000b23e8
    
    Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     2099199     1048576   83  Linux
    /dev/sda2         2099200    67108863    32504832   8e  Linux LVM
    ```
1. At the command prompt type
    ```
    fdisk /dev/sda
    ```
1. Type `p` to print the partition table and press Enter
1. Type `n` to add a new partition
1. Type `p` again to make it a primary partition
1. Now you'll be prompted to pick the first cylinder which will most likely come at the end of your last partition; **which is also listed as the default**.
1. If you want it to take up the rest of the space available, just **choose the default value for the last cylinder**.
1. Type `w` to save these changes
1. Reboot
1. Log back in as root
1. At the command prompt type, you'll notice another partition is present
    ```
    fdisk -l
    ```
    EG:
    ```
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
    ```
14. Add the physical volume to the existing volume group using the vgextend command.
    1. Find the name of the volume group.
        1. See Find Logical volumes (LVs) contained in Physical Volume (PVs) in LVM
    1. EG, the volume group is centos and the volume is root:
        ```
        lsblk
        NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        fd0               2:0    1    4K  0 disk
        sda               8:0    0  100G  0 disk
        ├─sda1            8:1    0    1G  0 part /boot
        ├─sda2            8:2    0   31G  0 part
        │ ├─centos-root 253:0    0 95.8G  0 lvm  /
        │ └─centos-swap 253:1    0  3.2G  0 lvm  [SWAP]
        └─sda3            8:3    0   68G  0 part
        sr0              11:0    1 1024M  0 rom
        ```
    1. Now type:
        ```
        vgextend [volume group] /dev/sdaX
        ```
        EG:
        ```
        vgextend centos /dev/sda3)
        ```
1. To find the amount of free space available on the physical volume type
    ```
    vgdisplay [volume group] | grep "Free"
    ```
    EG:
    ```
    vgdisplay centos | grep "Free"
    Free  PE / Size       17408 / 68 GiB
    ```
1. Extend the logical volume by the amount of free space shown in the previous step by typing
    ```
    lvextend  -L+[freespace]G /dev/volgroup/volume
    ```
    EG:
    ```
    lvextend -L+68G /dev/centos/root
    ```
1. Expand the filesystem

    1. If you have an ext3 filesystem you can expand it in the logical volume using the command
        ```
        resize2fs /dev/volgroup/volume
        ```
        EG:
        ```
        resize2fs /dev/centos/root
        ```
    1. If you have an xfs filesystem you can expand it in the logical volume using the command
        ```
        xfs_growfs /dev/volgroup/volume
        ```
        EG:
        ```
        xfs_growfs /dev/centos/root
        ```
1. You can now run the df command to verify that you have more space
    ```
    df -h
    ```
