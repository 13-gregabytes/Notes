# Installing Postgres on CentOS

## Install Postgres
Go to https://www.postgresql.org/download/linux/redhat/ to get the download steps

To use the PostgreSQL Yum Repository, follow these steps:

- Select version:
  - 9.6
- Select platform:
  - RedHat Enterprise, CentOS, Scientific or Oracle version 7
- Select architecture:
  - x86_64

Install the repository RPM:
```
yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```
### Install the client packages:
```
yum install postgresql96
```
### Install the server packages:
```
yum install postgresql96-server
```
### Initialize the database and enable automatic start:
```
/usr/pgsql-9.6/bin/postgresql96-setup initdb
systemctl enable postgresql-9.6
systemctl start postgresql-9.6
```

## Login as postgres user
### Add postgres bin to path
Modify /home/postgres/.bash_profile
```
PATH=$PATH:/usr/pgsql-9.6/bin/postgres
export PATH
```
### Create database
>https://www.postgresql.org/docs/9.6/tutorial-createdb.html
```
createdb chpdb
psql chpdb
```
### Create Role
>https://www.postgresql.org/docs/9.6/database-roles.html
```
CREATE ROLE chp LOGIN; (or CREATE USER name;)
SELECT rolname FROM pg_roles;
ALTER ROLE chp WITH PASSWORD 'chp';
```
### Alter pg_hba.conf
#### Modify `/var/lib/pgsql/9.6/data/pg_hba.conf`
This line:
`local all postgres peer`
Should be:
`local all postgres md5`

Append to bottom:
`postgres host chpdb chp all password`

#### Alter `postgresql.conf`
Uncomment the listen_addresses line and set its value to '*'
`listen_addresses = '*'`

## Restart postgres AS ROOT
```
systemctl restart postgresql-9.6
```