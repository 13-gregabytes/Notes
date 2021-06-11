# Adding a proxy to yum

### Edit `/etc/yum.conf`
```
proxy=http://proxyhost.domainname.net:3128
# below is optional
#proxy_username=<username>
#proxy_password=<password>
```

### List repo to verify internet connectivity
```
yum repolist
```