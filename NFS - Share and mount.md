# NFS - Share and mount

## Share mount from Linux (RHEL)
>You can only share mount points

### Is NFS started
```
lsmod | grep nfs
```
### Install nfs utilities
```
yum install nfs-utils
```
or
```
apt-get install nfs-common
```
### Start NFS
```
rpc.nfsd
```
### Tell Linux (RHEL) to share your mount
```
vi /etc/exports
```
>Format : {Directory to share} {IP of box to share with} ([read/write],[do a sync])

```
/ImasAssetStore 192.168.111.111(rw,sync)
/ImasAssetStore/sdd1/store2 192.168.111.111(rw,sync)
/ImasAssetStore/sdc1/store1 192.168.111.111(rw,sync)
```
### Export the filesystem
```
exportfs -a
```
### Restart the nfs server
```
/sbin/service nfs stop
/sbin/service nfs start
```
>/sbin/service nfs restart (should work as well)

### Give global permissions to the shared directories
>This may or may not be necessary
```
chmod -R 777 /ImasAssetStore/
```
##Mount an nfs share on Linux (RHEL)

### Install nfs utilities
```
yum install nfs-utils
```
or
```
apt-get install nfs-common
```
### Mount the share temporarily
>Format : mount -t nfs {computer sharing}:{name of share from dfstab} {mount directory}
```
mount -t nfs otsapr01:AssetStore/store1 /AssetStore/store1
```
### Mount the share permanently
>If the prior command was successful you can read/write the mount directories and sub directories then make the mount permanent.

```
vi /etc/fstab
```
>Format : {host computer}:{Host share point (from /etc/exports)} {Local mount point} nfs rw 0 0
```
otsapr01:/ImasAssetStore/sdd1/store2 /ImasAssetStore/sdd1/store2 nfs rw 0 0
```
### Make sure uid and gid match user on server
```
lslogins <username>
```
If they don’t, then change uid on client:
```
usermod -u <new uid> <username>
```
And change the gid:
```
groupmod –g <new gid> <old gid>
```
You must also change the gid on any files which have the old gid
```
find / -gid <old gid> ! -type l -exec chgrp <new gid> {} \;
```
