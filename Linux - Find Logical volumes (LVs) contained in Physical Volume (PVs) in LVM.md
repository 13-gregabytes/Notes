# Find Logical volumes (LVs) contained in Physical Volume (PVs) in LVM
>Found at https://www.thegeekdiary.com/centos-rhel-how-to-find-logical-volumes-lvs-that-are-part-of-a-physical-volume-pv-in-lvm/

## Using lsblk command
`lsblk` command gives a nice tree layout representation of disks/partitions and volumes residing on them.
```
# lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                     11:0    1 1024M  0 rom
sda                      8:0    0  250G  0 disk
├─sda1                   8:1    0  500M  0 part /boot
├─sda2                   8:2    0  187G  0 part
│ └─vg_os-lv_os (dm-0) 253:0 0 187G 0 lvm /
└─sda3                   8:3    0 62.5G  0 part [SWAP]
```
## Using lvs command
Another easy to use command is lvs. `lvs` with `-o +devices` arguments, we can find out the LV, VG and corresponding device used.
```
# lvs -a -o +devices
LV    VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices    
lv_os vg_os -wi-ao---- 187.01g                                                     /dev/sda2(0)
```
## Using pvdisplay command
Another handy command is `pvdisplay` with `-m` option. With the -m option we can display the logical volume(s) on the PV.
```
# pvs
PV         VG    Fmt  Attr PSize   PFree
/dev/sda2  vg_os lvm2 a--u 187.01g    0
# pvdisplay /dev/sda2 -m
--- Physical volume ---
PV Name               /dev/sda2
VG Name               vg_os
PV Size               187.01 GiB / not usable 3.00 MiB
Allocatable           yes (but full)
PE Size               4.00 MiB
Total PE              47874
Free PE               0
Allocated PE          47874
PV UUID               I57oVs-dxyE-ofLR-AuTO-WdqU-F8lU-2fD7wS

--- Physical Segments ---
Physical extent 0 to 47873:
Logical volume /dev/vg_os/lv_os
Logical extents 0 to 47873
```
## Using vgdisplay command
Here we will have all physical devices used per Volume Group, not per Logical Volume. So first we have the VG info, below we will find all LVs which corresponds to its VG, and after we will see all PVs attached in our VG.
```
# vgdisplay -v
    Using volume group(s) on command line.
--- Volume group ---
VG Name               vg_os
System ID            
Format                lvm2
Metadata Areas        1
Metadata Sequence No  2
VG Access             read/write
VG Status             resizable
MAX LV                0
Cur LV                1
Open LV               1
Max PV                0
Cur PV                1
Act PV                1
VG Size               187.01 GiB
PE Size               4.00 MiB
Total PE              47874
Alloc PE / Size       47874 / 187.01 GiB
Free  PE / Size       0 / 0  
VG UUID               R6fvJR-Ev2s-VAFZ-Vdg0-2IhR-cY42-Bunqnx

--- Logical volume ---
LV Path /dev/vg_os/lv_os
LV Name                lv_os
VG Name                vg_os
LV UUID                Ifl0gt-DYCP-JVMW-yUJh-K79e-hh1t-D63Djv
LV Write Access        read/write
LV Creation host, time VOM-VCS-MONITOR, 2014-08-15 21:58:01 +0530
LV Status              available
# open                 1
LV Size                187.01 GiB
Current LE             47874
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     256
  Block device           253:0

--- Physical volumes ---
PV Name /dev/sda2    
PV UUID               I57oVs-dxyE-ofLR-AuTO-WdqU-F8lU-2fD7wS
PV Status             allocatable
Total PE / Free PE    47874 / 0
```
## Using lvdisplay command
With the `–maps` argument, `lvdisplay` command will list out all the logical volumes in the system along with their physical volumes.
```
# lvdisplay --maps
--- Logical volume ---
LV Path /dev/vg_os/lv_os
LV Name                lv_os
VG Name                vg_os
LV UUID                Ifl0gt-DYCP-JVMW-yUJh-K79e-hh1t-D63Djv
LV Write Access        read/write
LV Creation host, time VOM-VCS-MONITOR, 2014-08-15 21:58:01 +0530
LV Status              available
# open                 1
LV Size                187.01 GiB
Current LE             47874
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     256
  Block device           253:0

--- Segments ---
Logical extents 0 to 47873:
Type        linear
Physical volume /dev/sda2
Physical extents    0 to 47873
```