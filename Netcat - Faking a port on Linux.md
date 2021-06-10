# Netcat - Faking a port on Linux

## Fake a port listener on Unix.
use netcat (nc) with the `k` and `l` arguments. You will also need to specify a port. I like 9999 for this.

Command arguments

|arg|desc|
|---|---|
|-k|Forces nc to stay listening for another connection after its current connection is completed. It is an error to use this option without the -l option.|
|-l|Used to specify that nc should listen for an incoming connection rather than initiate a connection to a remote host. It is an error to use this option in conjunction with the -p , -s , or -z options. Additionally, any timeouts specified with the -w option are ignored.|

## Example
```
nc –kl 9999
```