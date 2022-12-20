# OpenLDAP HowTo

## Install and Configure OpenLDAP on Ubuntu

1. Ensure OpenLDAP server host has a properly configured hostname:
```commandline
hostnamectl set-hostname ldap.nhadzter.local
```

2. Add the entry to /etc/hosts ( this is for testing purposes if you don't have access to a name server )
```commandline
192.168.56.254 ldap.nhadzter.local ldap
```
3. Install OpenLDAP
```commandline
apt -y install slapd ldap-utils ldap-common
```

Note: An admin password will be asked during installation, set the password and proceed with the installation

4. Create Base DN for Users and Groups.
   1. Create the file base.ldif with the following content:
   ```commandline
   dn: ou=people,dc=nhadzter,dc=local
   objectClass: organizationalUnit
   ou: people

   dn: ou=groups,dc=nhadzter,dc=local
   objectClass: organizationalUnit
   ou: groups
   ```
   2. Using ldapadd utility, add the Base DN:
   ```commandline
   ldapadd -x -D cn=admin,dc=nhadzter,dc=local -W -f base.ldif
   ```

5. Add User accounts.
   1. Create the file users.ldif with the following content
   ```commandline
   dn: uid=nhadie,ou=people,dc=nhadzter,dc=local
   objectClass: inetOrgPerson
   objectClass: posixAccount
   objectClass: shadowAccount
   cn: nhadie
   sn: monio
   userPassword: {SSHA}vE0J8mV4wZyoShexlKG3SM8zyR6t/kNP
   loginShell: /bin/bash
   uidNumber: 8000
   gidNumber: 8000
   homeDirectory: /home/nhadie  
   ```
   Note:
   - If integrating Linux hosts as LDAP clients, ensure users contain POSIX attributes.
   - Ensure to use high values on uid and gid to avoid conflict with local users.
   - userPassword is the hashed password set for the user, which can be generated using different tools. In this documentation we will use slappasswd:
   ```commandline
   root@ldap:~# slappasswd 
   New password: 
   Re-enter new password: 
   {SSHA}vE0J8mV4wZyoShexlKG3SM8zyR6t/kNP
   ```
   2. Using ldapadd utility, create the user/s:
   ```commandline
   ldapadd -x -D cn=admin,dc=nhadzter,dc=local -W -f users.ldif
   ```
   
6. Add Groups
   1. Create the file groups.ldif with the following content
   ```commandline
   dn: cn=nhadie,ou=people,dc=nhadzter,dc=local
   objectClass: posixGroup
   cn: nhadie
   gidNumber: 8000
   memberUid: nhadie     
   ```
   2. Using ldapadd utility, create the group/s:
   ```commandline
   ldapadd -x -D cn=admin,dc=nhadzter,dc=local -W -f groups.ldif
   ```

## Configure Ubuntu as LDAP Client

1. Ensure /etc/hosts has the entry for the ldap server
```commandline
192.168.56.254 ldap.nhadzter.local ldap
```
2. Install ldap client packages:
```commandline
apt -y install libnss-ldap libpam-ldap ldap-utils
```
Note: Installation will prompt for LDAP endpoints, Base DN and admin password.
3. After installation, update /etc/nsswitch.conf to use ldap authentication:
```commandline
passwd:         files systemd ldap
group:          files systemd ldap
```
4. Update /etc/pam.d/common-password remove use_authtok after pam_ldap.so
5. Update /etc/pam.d/common-session to enable home directory creation on first time login:
```commandline
session optional pam_mkhomedir.so skel=/etc/skel umask=077
```
6. Test authentication by switching to the users created in the ldap server
```commandline
root@client:~# su - nhadie
Creating directory '/home/nhadie  '.
nhadie@client:~$ pwd
/home/nhadie
nhadie@client:~$ id
uid=8000(nhadie) gid=8000(nhadie) groups=8000(nhadie)
```


