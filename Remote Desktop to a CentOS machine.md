# Remote Desktop to a CentOS machine

>https://arstech.net/centos-7-remote-desktop-from-windows/
```
yum -y install epel-release
```
```
yum -y install xrdp tigervnc-server
```
```
systemctl start xrdp.service
```
```
systemctl enable xrdp.service
```
```
firewall-cmd --permanent --zone=public --add-port=3389/tcp
```
```
firewall-cmd --reload
```