# Debug a java application running on the command line

Use the arguments `-Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=8000`

### Command arguments
|arg|desc|
|---|---|
|-Xdebug|Tells the java application to wait for a debugger to attach.|
|-Xrunjdwp:transport=dt_socket,server=y,address=8000|The only parameter you should change here is the address. The port needs to be open on your machine so I defaulted to 8000 as this is usually open.|

### Example
```
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=8000 –jar CHP_4.10.3_rc1a_for_jBoss.jar
```