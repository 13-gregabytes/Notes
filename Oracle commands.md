# Oracle commands

## Starting Oracle
### Log into sqlplus, connect to the main db and start it up
```
sqlplus /nolog
conn / as sysdba
STARTUP
```
### Exit sqlplus and start the listener
```
lsnrctl start
```
## Stopping Oracle
### Stop the listener
```
lsnrctl stop
```
### Log into sqlplus, connect to the main db and shut it do
```
sqlplus /nolog
conn / as sysdba
SHUTDOWN [normal|immediate|abort]
```
>In a **normal shutdown**, Oracle Database waits for all currently-connected users to disconnect and disallows any new connections before shutting down. This is the default mode.

>In an **immediate shutdown**, Oracle Database terminates and rolls back active transactions, disconnects clients, and shuts down.

>In an **abort shutdown**, Oracle Database terminates active transactions and disconnects users; it does not roll back transactions. The database performs automatic recovery and rollback the next time it is started. Use this mode only in emergencies.

## List table sizes
```
SELECT DS.TABLESPACE_NAME, SEGMENT_NAME, SUM(DS.BYTES)
FROM DBA_SEGMENTS DS
WHERE SEGMENT_NAME IN (SELECT TABLE_NAME FROM DBA_TABLES)
GROUP BY DS.TABLESPACE_NAME, SEGMENT_NAME
ORDER BY DS.TABLESPACE_NAME, SUM(DS.BYTES) DESC, SEGMENT_NAME;
```
## List tablespaces
```
SELECT * FROM v$TABLESPACE;
```
## See datafiles being used
```
SELECT name, bytes/1024/1024 AS size_mb FROM v$datafile;
```
## Add a Datafile
```
ALTER TABLESPACE DEVELOPMENT ADD DATAFILE '/u04/app/oracle/oradata/StAlbans/development03.dbf' size 10240m autoextend on maxsize 32767m;
```
## Clear up Audit tables
https://dba.stackexchange.com/questions/155288/how-to-reduce-sysaux-tablespace-size-with-audit-trail-unified-enabled
```
SQL> select owner||'."'||table_name||'"' t from dba_tables where table_name like 'CLI_SWP$%';

T
---------------------------------------------------------------------------------------------
AUDSYS."CLI_SWP$fea2ec1e$1$1"
```
```
SQL>  shutdown immediate;

Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup upgrade
ORACLE instance started.

Total System Global Area 7315603456 bytes
Fixed Size                  2305328 bytes
Variable Size            3657433808 bytes
Database Buffers         3640655872 bytes
Redo Buffers               15208448 bytes
Database mounted.
Database opened.
```
```
SQL> truncate table AUDSYS."CLI_SWP$fea2ec1e$1$1";

Table truncated.
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area 7315603456 bytes
Fixed Size                  2305328 bytes
Variable Size            3657433808 bytes
Database Buffers         3640655872 bytes
Redo Buffers               15208448 bytes
Database mounted.
Database opened.
```
## Change password expiry time
Logon to the product database as the SYSTEM user (not the application administration user).
```
sqlplus /nolog
conn / as sysdba
```
Find the profile that has the PASSWORD_LIFE_TIME set to anything but UNLIMITED
```
select * from dba_profiles where RESOURCE_NAME LIKE 'PASSWORD_LIFE_TIME';
```
Alter the profiles with the following statement:
```
ALTER PROFILE profile_name LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```
>profile_name is the name of the profile where your need to set the password life to UNLIMITED.

>You may need to change the Oracle user's password before this take effect.

## Change password
```
sqlplus> password
Changing password for <user>
```
```
Old password: OldPassword
New password: NewPassword
Retype new password: NewPassword
```
>Replace OldPassword and NewPassword with your old and new passwords