# How to install svn on CentOS 
```
vi /etc/yum.repos.d/wandisco-svn.repo


[WandiscoSVN]
name=Wandisco SVN Repo
baseurl=http://opensource.wandisco.com/centos/$releasever/svn-1.8/RPMS/$basearch/
enabled=1
gpgcheck=0
```
```
yum remove subversion*
```
```
yum clean all
```
```
yum install subversion
```