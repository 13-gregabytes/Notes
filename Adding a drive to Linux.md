# Adding a drive to Linux
## Debian
###Find the disk you added
>(warning) You may need to restart the machine before the drive is visible.
```
fdisk -l
```
Sample output:
```
Disk /dev/sda: 21.4 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes


Device Boot Start End Blocks Id System
/dev/sda1 * 1 2517 20217771 83 Linux
/dev/sda2 2518 2610 747022+ 5 Extended
/dev/sda5 2518 2610 746991 82 Linux swap / Solaris

Disk /dev/sdb: 32.2 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

Disk /dev/sdb doesn't contain a valid partition table
```
### Partition the new drive
```
fdisk /dev/sdb
New -> Primary -> Specify size in MB
(W)rite
(Q)uit
```
### Format the disk (ext3)
```
mkfs.ext3 /dev/sdb1
```
### Make a directory to mount to
```
mkdir <new working directory>
```
### Mount the device to the directory
```
mount /dev/dsk/sdb1 <new working directory>
```

### If the mount was successful unmount so we can modify `/etc/fstab` and remount with the auto command.
```
umount <new working directory>
```
### Edit fstab to remount at boot time
```
vi /etc/fstab
```
Copy an existing line and modify it to fit your disk
```
/dev/sdb1 /u01 ext3 defaults,errors=remount-ro 0 1
```
### Mount all devices in fstab
```
mount -a
```
