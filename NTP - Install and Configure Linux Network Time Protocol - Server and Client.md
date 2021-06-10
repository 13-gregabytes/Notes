# NTP - Install and Configure Linux Network Time Protocol - Server and Client

## Install NTP Server
### Install NTP
#### Debian
```
sudo apt-get install ntp ntpdate ntp-doc
```
#### Redhat/CentOS
```
yum install ntp ntpdate ntp-doc
```
### Set ntp to start on machine startup
`systemctl enable ntpd`
or
`chkconfig ntpd on`

### Setup Restrict values in ntp.conf
Modify the `/etc/ntp.conf` file to make sure it has the following two restrict lines.
```
vi /etc/ntp.conf
```
```
# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
```
The first restrict line allows other clients to query your time server. This restrict line has the following parameters

|parameter|description|
|---|---|
|noquery|prevents dumping status data from ntpd.|
|notrap|prevents control message trap service.|
|nomodify|prevents all ntpq queries that attempts to modify the server.|
|nopeer|prevents all packets that attempts to establish a peer association.|
|kod|Kiss-o-death packet is to be sent to reduce unwanted queries|
|-6|forces the DNS resolution to the IPV6 address resolution.|

>For more information on the access parameters list, Please refer to documentation on “man ntp_acc”

### Allow Only Specific Clients
To only allow machines on your own network to synchronize with your NTP server, add the following restrict line to your /etc/ntp.conf file:
```
vi /etc/ntp.conf
```
```
restrict 192.168.111.0 mask 255.255.255.0 nomodify notrap
```
If the localhost needs to have the full access to query or modify, add the following line to /etc/ntp.conf
```
vi /etc/ntp.conf
```
```
restrict 127.0.0.1
```
### Add Local Clock as Backup
Add the local clock to the ntp.conf file so that if the NTP server is disconnected from the internet, NTP server provides time from its local system clock.
```
vi /etc/ntp.conf
```
```
server  127.0.0.1 # local clock
fudge   127.0.0.1 stratum 10
```
In the above line, Stratum is used to synchronize the time with the server based on distance.

|stratum|description|
|---|---|
|stratum-0|always used as reference clock.|
|stratum-1|time server acts as a primary network time standard.|
|stratum-2|server is connected to the stratum-1 server over the network. Thus, a stratum-2 server gets its time via NTP packet requests from a stratum-1 server.|
|stratum-3|server gets its time via NTP packet requests from a stratum-2 server, and so on.|

### Setup NTP Log Parameters
Specify the drift file and the log file location in your ntp.conf file
```
vi /etc/ntp.conf
```
```
driftfile /var/lib/ntp/ntp.drift
logfile /var/log/ntp.log
```
driftfile is used to log how far your clock is from what it should be, and slowly ntp should lower this value as time progress.

### Start the NTP Server
After setting up appropriate values in the ntp.conf file, start the ntp service:

`service ntpd start`
or
`/etc/init.d/ntpd restart`

## Configure NTP Client to Synchronize with NTP Server
>This setup should be done on your NTP Client (Not on NTP-server)

### Edit your ntp.conf
>Optional

To synchronize the time of your local Linux client machine with NTP server, edit the /etc/ntp.conf file on the client side. Here is an example of how the sample entries looks like. In the following example, you are specifying multiple servers to act as time server, which is helpful when one of the timeservers fails.
```
vi /etc/ntp.conf
```
```
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst
```

**iburst**: After every poll, a burst of eight packets is sent instead of one. When the server is not responding, packets are sent 16s interval. When the server responds, packets are sent every 2s.

#### Edit your ntp.conf to reflect appropriate entries for your own NTP server.
```
vi /etc/ntp.conf
```
```
server 192.168.333.333 prefer
```
**prefer**: If this option is specified that server is preferred over other servers. A response from the preferred server will be discarded if it differs significantly different from other server’s responses.

### Start the NTP Daemon
Once the ntp.conf is configured with correct settings, start the ntp daemon.

`/etc/init.d/ntp start`
or
`service ntpd start`

You will see the NTP will slowly start to synchronize the time of your linux machine with the NTP Server.

### Set ntp to start on machine startup
`systemctl enable ntpd`
or
`chkconfig ntpd on`

### Check the NTP Status
Check the status of NTP using the ntpq command. If you get any connection refused errors then the time server is not responding or the NTP daemon/port is not started or listening.
```
ntpq -p
```
```
remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
fusebox.domainn 192.168.111.111   2 u   26   64    1    0.596  12614.2   0.000
```

### Set Local Date and Time
The `ntpdate` command can be used to set the local date and time by polling the NTP server. Typically, you’ll have to do this only one time.

Your jitter value should be low, else check the drift from the clock in the driftfile. This command synchronizes the time with your NTP server manually.
```
ntpdate –u 192.168.333.333
```

After this initial sync, NTP client will talk to the NTP server on an on-going basis to make sure the local time reflects the accurate time.

You can also use the following command to get the current status of ntpd.
```
ntpdc -c sysinfo
```
```
system peer:          0.0.0.0
system peer mode:     client
leap indicator:       11
stratum:              4
precision:            -23
root distance:        0.00279 s
root dispersion:      0.06271 s
reference ID:         [192.168.333.333]
reference time:       d70bd07b.f4b5cf2b  Wed, Apr 30 2014 15:41:47.955
system flags:         auth monitor ntp kernel stats
jitter:               0.000000 s
stability:            0.000 ppm
broadcastdelay:       0.000000 s
authdelay:            0.000000 s
```
### Check if ntpd is configured to run at system start
```
chkconfig --list ntpd
```
```
ntpd              0:off 1:off 2:on  3:on  4:on  5:on  6:off
```
or
```
systemctl list-unit-files | grep ntp
```
```
ntpd.service                                enabled
ntpdate.service                             disabled
```
### To obtain a brief status report from ntpd
```
ntpstat
```
```
unsynchronised
   polling server every 64 s
```
```
ntpstat
```
```
synchronised to NTP server (192.168.222.222) at stratum 2
   time correct to within 52 ms
   polling server every 1024 s
```

## Appendix
### fusebox /etc/ntp.conf (server)
```
driftfile /var/lib/ntp/ntp.drift

statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable

server 0.uk.pool.ntp.org iburst
server 1.uk.pool.ntp.org iburst
server 2.uk.pool.ntp.org iburst
server 3.uk.pool.ntp.org iburst

restrict -4 default kod notrap nomodify nopeer noquery
restrict -6 default kod notrap nomodify nopeer noquery

restrict 192.168.111.0 mask 255.255.255.0 nomodify notrap

restrict 127.0.0.1
restrict ::1
```

### grazebox /etc/ntp.conf (client)
```
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery

restrict 127.0.0.1
restrict -6 ::1

server 192.168.111.222

driftfile /var/lib/ntp/drift

includefile /etc/ntp/crypto/pw

keys /etc/ntp/keys
```