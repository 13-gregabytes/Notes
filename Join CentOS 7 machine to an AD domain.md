# Join CentOS 7 machine to an AD domain
>Found these instructions at: https://www.rootusers.com/how-to-join-centos-linux-to-an-active-directory-domain/

## Preparing CentOS

First we want to install all of the below packages in CentOS.
```
yum install sssd realmd oddjob oddjob-mkhomedir adcli samba-common samba-common-tools krb5-workstation openldap-clients policycoreutils-python -y
```
The CentOS server will need to be able to resolve the Active Directory domain in order to successfully join it.

In this instance my DNS server in `/etc/resolv.conf` is set to one of the Active Directory servers hosting the example.com domain that I wish to join.
```
[root@centos7 ~]# cat /etc/resolv.conf
search example.com
nameserver 192.168.1.2
```
Reboot the machine

`shutdown -r now` or `reboot`

## Join CentOS To Windows Domain
Now that we’ve got that out of the way we can actually join the domain, this can be done with the ‘realm join’ command as shown below.

You will need to specify the username of a user in the domain that has privileges to join a computer to the domain.
```
[root@centos7 ~]# realm join --user=administrator example.com
Password for administrator:
```
Once you enter the password for your specific account, the `/etc/sssd/sssd.conf` and `/etc/krb.conf` files will be automatically configured.

This is really great as editing these manually usually leads to all sorts of trivial problems when joining the domain. The `/etc/krb5.keytab` file is also created during this process.

If this fails, you can add `-v` to the end of the command for highly verbose output, which should give you more detailed information regarding the problem for further troubleshooting.

We can confirm that we’re in the realm (Linux terminology for the domain) by running the `realm list` command, as shown below.
```
[root@centos7 ~]# realm list
example.com
type: kerberos
realm-name: EXAMPLE.COM
domain-name: example.com
configured: kerberos-member
server-software: active-directory
client-software: sssd
required-package: oddjob
required-package: oddjob-mkhomedir
required-package: sssd
required-package: adcli
required-package: samba-common-tools
login-formats: %U@example.com
login-policy: allow-realm-logins
```
Once this has completed successfully, a computer object will be created in Active Directory in the default computers container.

Now that our Linux server is a member of the Active Directory domain we can perform some tests. By default if we want to specify any users in the domain, we need to specify the domain name. For example with the `id` command below, we get nothing back for ‘administrator’, however ‘administrator@example.com’ shows the UID for the account as well as all the groups the account is a member of in the Active Directory domain.
```
[root@centos7 ~]# id administrator
id: administrator: no such user
```
```
[root@centos7 ~]# id administrator@example.com
uid=1829600500(administrator@example.com) gid=1829600513(domain users@example.com) groups=1829600513(domain users@example.com),1829600512(domain admins@example.com),1829600572(denied rodc password replication group@example.com),1829600519(enterprise admins@example.com),1829600518(schema admins@example.com),1829600520(group policy creator owners@example.com)
```
We can change this behaviour by modifying the `/etc/sssd/sssd.conf` file, the following lines need to change from:
```
use_fully_qualified_names = True
fallback_homedir = /home/%u@%d
```
To the below, which does not require the fully qualified domain name (FQDN) to be specified. This also modifies the user directory in `/home` from having the FQDN specified after the username.
```
use_fully_qualified_names = False
fallback_homedir = /home/%u
```
To apply these changes, restart sssd.
```
[root@centos7 ~]# systemctl restart sssd
```
Now we should be able to find user accounts without specifying the domain, as shown below this now works where it did not previously.
```
[root@centos7 ~]# id administrator
uid=1829600500(administrator) gid=1829600513(domain users) groups=1829600513(domain users),1829600512(domain admins),1829600572(denied rodc password replication group),1829600520(group policy creator owners),1829600519(enterprise admins),1829600518(schema admins)
```
If this is still not correctly working for you, I suggest that you take a look at flushing your sssd cache.

## Configuring SSH and Sudo Access
Now that we have successfully joined our CentOS server to the example.com domain, we can SSH in as any domain user from Active Directory with default settings.
```
[root@centos7 ~]# ssh user1@localhost
user1@localhost's password:
Creating home directory for user1.
```
We can further restrict SSH access by modifying the /etc/ssh/sshd_config file and make use of things like AllowUsers or AllowGroups to only allow certain user or groups from AD to have access.

See our guide to the sshd_config file for further information.

Don’t forget to restart sshd if you make any changes to this file in order to apply them.

We can also modify our sudoers configuration to allow our user account from the domain the desired level of access. I usually create an Active Directory group called something like ‘sudoers’, put my user in it, then allow this group sudo access by creating a file in /etc/sudoers.d/ which allows root access to be centrally controlled by AD.

Below is an example of this, the ‘sudoers’ group will have full root access.
```
[root@centos7 ~]# cat /etc/sudoers.d/sudoers
%sudoers ALL=(ALL) ALL
```
This group only exists in Active Directory, our Linux server can see that user1 is a member of the sudoers group in Active Directory, and respects this group configuration and allows user1 root privileges as per the above configuration.

With this in place, our user1 account in the example.com Active Directory domain will now be able to use the sudo command to run commands with root privileges.
```
[user1@centos7 ~]$ sudo su
[sudo] password for user1:
[root@centos7 user1]#
[root@centos7 user1]# whoami
root
```
That’s all there is to it, we can now SSH to a Linux server with a user account from our Active Directory domain and even grant specific users or groups from AD specific levels of access.

## Leaving The Domain
If you want to reverse the process and remove yourself from the domain, simply run the ‘realm leave’ command followed by the domain name, as shown below.
```
[root@centos7 ~]# realm leave example.com
```
This will complete without any further user input. It will delete the computer object that was created in Active Directory, remove the keytab file, and set the sssd.conf and krb5.conf files back to default.