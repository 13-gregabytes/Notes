# Close a file via file descriptor from lsof
Use `lsof -p $PID` and find the file descriptor (4th column)
```
root@blah:~# lsof -p 1737 | grep "(deleted)"
```
```
apache2 1737 root 6w REG 0,25 0 207401

(deleted)/var/log/apache2/other_vhosts_access.log
```

>4th column is 6w, meaning file descriptor 6 and it was opened for writing (w).

Then:
```
gdb -p $PID

p close($FD)
```

eg:
```
gdb -p 1737

.....

(gdb) p close(6)

$1 = 0

...

q

Quit anyway? (y or n) y
```
```
Detaching from program: /usr/lib/apache2/mpm-prefork/apache2, process 1737
```