# Linux - Change UID and GID

>These instructions were found at: http://www.cyberciti.biz/faq/linux-change-user-group-uid-gid-for-all-owned-files/
### Find the user's UID in the passwd file
```
grep auser /etc/passwd
```
```
auser:x:1000:1000:auser:/home/auser:/bin/bash
```

>Note that all files which are located in the user’s home directory will have the file UID changed automatically as soon as you execute these two command. However, files outside user’s home directory need to be changed manually.

### Assign a new UID to user using the usermod command.
```
usermod -u 1000 auser
```
### Assign a new GID to group using the groupmod command.
```
groupmod -g 1000 auser
```
### Use the chown and chgrp commands to change old UID and GID.
>You can automate this with the help of find command.
```
find / -group 500 -exec chgrp -h auser {} \;
```
```
find / -user 500 -exec chown -h auser {} \;
```
> The -exec command executes chgrp or chown command on each file. The -h option passed to the chgrp/chown command affects each symbolic link instead of any referenced file.