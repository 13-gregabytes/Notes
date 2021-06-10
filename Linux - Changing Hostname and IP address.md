# Linux - Changing Hostname and IP address

## CentOS
Configure network interface (eg, eth0)
```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
```
DEVICE="eth0"

NM_CONTROLLED="yes"
ONBOOT=yes
HWADDR=A4:BA:DB:37:F1:04
TYPE=Ethernet
BOOTPROTO=static
NAME="System eth0"
UUID=5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03
IPADDR=192.168.111.111
NETMASK=255.255.255.0
```
Configure hostname and default gateway
```
vi /etc/sysconfig/network
```
```
NETWORKING=yes

HOSTNAME=centos6

GATEWAY=192.168.111.111
```
```
vi /etc/hostname
```
```
HOSTNAME=centos6
```
Configure DNS Server
```
vi /etc/resolv.conf
```
```
nameserver 8.8.8.8 # Replace with your nameserver ip

nameserver 192.168.111.111 # Replace with your nameserver ip
```
Restart Network Interface
```
/etc/init.d/network restart
```

## Debian
Configure network interface (eg, eth0)
```
vi /etc/network/interfaces
```
For static IP:
```
auto eth0
iface eth0 inet static
address 192.168.111.111
netmask 255.255.255.0
network 192.168.111.0
broadcast 192.168.111.255
gateway 192.168.111.1
```
For dynamic IP:
```
auto eth0
iface eth0 inet dhcp
allow-hotplug eth0
Configure hostname
vi /etc/hostname
HOSTNAME=centos6
```
Edit hosts file
```
vi /etc/hosts
```
```
127.0.0.1 centos6
```
Restart Network Interface
```
invoke-rc.d hostname.sh start
invoke-rc.d networking force-reload
invoke-rc.d network-manager force-reload
```