# Haproxy - Setting up
>Helpful tips:  
https://ops.tips/blog/haproxy-http2/  
https://www.haproxy.com/blog/haproxy-2-0-and-beyond/

## Installing haproxy:
Version 2.0 (latest LTS)
### install prereqs
```
yum install -y gcc make openssl-devel pcre-devel readline-devel systemd-devel tar
```
### make haproxy
```
make TARGET=linux-glibc USE_OPENSSL=1 USE_PCRE=1 USE_SYSTEMD=1 USE_ZLIB=1
make install
```
### create some needed directories
```
ls -l /var/lib/haproxy/
mkdir -p /etc/haproxy
mkdir -p /var/lib/haproxy
touch /var/lib/haproxy/stats
```
### link to sbin so other users can start haproxy
```
ln -s /usr/local/sbin/haproxy /usr/sbin/haproxy
```
### check config (-q will not out put warnings, only errors)
```
haproxy -f /etc/haproxy/haproxy.cfg -c -q
```
### start haproxy in daemon mode
```
haproxy -f /etc/haproxy/haproxy.cfg -D
```
### start haproxy in debug mode
```
haproxy -db -f /etc/haproxy/haproxy.cfg
```

## Configuring haproxy:
http://haproxy.1wt.eu/download/1.5/doc/configuration.txt

http://www.rackspace.com/knowledge_center/article/setting-up-haproxy

### Version 2.0

/etc/haproxy/haproxy.cfg
```
global
log 127.0.0.1  local0 debug
log 127.0.0.1  local1 debug
maxconn 4096
# uid 99
# gid 99
daemon
# debug
# quiet

defaults
log        global
mode       http
option     httplog
option     dontlognull
retries    3
option     redispatch
maxconn    2000
contimeout 50000
clitimeout 50000
srvtimeout 50000

    stats enable
    stats hide-version
     
    stats scope chp_backend
 
    stats uri /haproxy?stats
    stats realm Haproxy\ Statistics
    stats auth admin:Adm1nn

########## STATISTIC COLLECTING ######################################
listen stats
bind fuselab:7828
option httpclose
######################################################################


########### CHP HTTPS #################################################
frontend chp_https
bind fuselab:8443 ssl crt /etc/haproxy/certs/certs-combo.pem alpn h2,http/1.1
reqadd X-Forwarded-Proto:\ https
default_backend chp_backend

backend chp_backend
option forwardfor
option forwardfor except fuselab
option log-health-checks

#    option httpchk HEAD /CHP/FrontController_Info?_picdar_event=APPMAN_Availability HTTP/1.1

    balance roundrobin
 
    timeout server 7200000
 
    cookie WEBSERVERID insert
 
    server grazelab-8443 grazelab.domainname.net:8443 ssl verify none check inter 10 alpn h2
    server grazelab-8543 grazelab.domainname.net:8543 cookie chpapp8180 ssl verify none check inter 5000 alpn h2
#######################################################################
```
## Set up haproxy to log to via syslog
>(warning) This is for Debian. RedHat will most likely be different.

ref: http://kvz.io/blog/2010/08/11/haproxy-logging/

### Create /etc/rsyslog.d/haproxy.conf

>(warning) Otherwise consider putting these two in /etc/rsyslog.conf instead:
```
$ModLoad imudp
$UDPServerAddress 127.0.0.1
$UDPServerRun 514
```

>(warning) In any case, put these two in /etc/rsyslog.d/haproxy.conf:
```
local0.* -/var/log/haproxy.log
& ~
# & ~ means not to put what matched in the above line anywhere else for the rest of the rules
# http://serverfault.com/questions/214312/how-to-keep-haproxy-log-messages-out-of-var-log-syslog
```
###Create /etc/logrotate.d/haproxy
```
/var/log/haproxy
{
rotate 4
weekly
missingok
notifempty
delaycompress
compress
postrotate
invoke-rc.d rsyslog reload > /dev/null
endscript
}
```

### Restart rsyslog
`invoke-rc.d rsyslog restart`
or
`/etc/init.d/rsyslog restart`
