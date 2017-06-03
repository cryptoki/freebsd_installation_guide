## User Management
### Basic System
1. root
   
   the root user should only accessible from the localhost
2. ssh access user, in the script I call the user 'userSsh'
   * it's a user with normal shell, I prefer csh 
   * it's in the group 'userSsh'
   * it's in the additional group wheel, to be allowed to get root (su -) rights
   * the only and really the only use for the user is to get root rights
   * for security reasons we will configure this user in sshd_conf as the only user to login at the sshd
   
Now we have two user for the basic system. 

### The Challenge
I plan to add few more users for domain hosting, I will call the user
   1. *domainUser1*
   2. *domainUser2*
   3. *...*
   
The user directory should be on the host system. I prefer the host system, because
   * the user directory */usr/home/domainUserN* is only for storage data
   * the user has *NO* login rights on the host system
   * the user has the login group *nologin*
   * the user home *OR* parts of the user home can easily mounted on the jail / guest system
   * the user home *OR* parts of the user home can easily make read only for a jail / guest system and with unionfs it can be mixed up. read only and writeable in a owned storage part
   
### My Solution / One Solution
#### Create User and Group with own UID and GID on the host system
  1. domainUser1 with group domainUser1
     * UID 10.001
     * GID 10.001
  2. domainUser2 with group domainUser2
     * UID 10.002
     * GID 10.002
     
I use ID above 10.000 to avoid ID clashes. The ID must be below 32768. I need only few users for my own purpose. I can create the users manually.

#### Create Users and Group on the Jail/Guest System
create the *same* users and groups on the guest/jail system with the same UID/GID an the Jail/Guest system
* you need only to create the user on the jail if u need to mount the user home or a part of the user home in the jail
* mount the necessary directory in the jail
* if a service process like nginx needs to read or write the user data, add the nginx user on the jail/guest system to the user group. The nginx user has its own nginx group and all additional user groups. 
     



### Increase security
Change the default umask in the */etc/login.conf* from *022* to *027*. The permissions would grant full access to users, limited (read-only) access to groups and no rights for other users.
```
default:\
        :passwd_format=sha512:\
        :copyright=/etc/COPYRIGHT:\
        ...
        ...
        ...
        :umask=027:
```
rebuild database
```
# cap_mkdb /etc/login.conf
```

for the root user, u have to change it in the *.cshrc* in the root home directory.
```
# A righteous umask
umask 27
```
logout and login again.


#### what does it mean?
```
# touch index.html # with umask 022
# ls -la
-rw-r--r--  1 domainUser1  domainUser1  10 Jun  3 13:12 index.html

# touch index.html # with umask 027
# ls -la
-rw-r-----  1 domainUser1  domainUser1  10 Jun  3 13:12 index.html

```
RTFM ;-) https://www.cyberciti.biz/tips/understanding-linux-unix-umask-value-usage.html



### Setup
#### Groups
```
pw groupadd -g 10001 -n domainUser1
pw groupadd -g 10002 -n domainUser2
pw groupadd -g 10003 -n domainUser3

pw group show -a
```

#### Users
Use the adduser command to create new user. 
```
adduser
# 1 The username and the group name must be identical. 
# 2 The User ID UID is identical with the group id GID (must not, but I use the same)
# 3 The login should be *nologin* .. the domain user has no login/shell rights
```

#### Create directory structure
```
# su -m domainUser1
```
change from user to the domainUser1. 

```
# cd /home/domainUser1
# mkdir www
# cd www
# mkdir www.meineDomain.de
# cd www.meineDomain.de
# mkdir public_html
# mkdir log
```

#### Change access rights for /home
```
root@wallace:/home # ls -la
total 51
drwxr-x--x   6 root            wheel           6 Jun  3 00:01 .
drwxr-xr-x  16 root            wheel          16 May 27 15:25 ..
drwxr-x---   2 sshAccessUser   sshAccessUser  11 Jun  3 13:25 sshAccessUser
drwxr-x---   3 domainUser1     domainUser1    11 Jun  3 13:56 domainUser1
drwxr-x---   3 domainUser2     domainUser2    11 Jun  3 00:25 domainUser2
drwxr-x---   3 domainUser3     domainUser3    11 Jun  3 00:07 domainUser3
```

```
root@wallace:/home/tour-report/www/www.meineDomain.de # ls -la
total 2
drwxr-x---  4 root  domainUser1  4 Jun  3 00:09 .
drwxr-x---  3 root  domainUser1  3 Jun  3 00:07 ..
drwxr-x---  2 root  domainUser1  2 Jun  3 00:09 log
drwxr-x---  2 root  domainUser1  4 Jun  3 13:26 public_html
```
the user has no free write rights on the www directory tree.

### OpenLDAP
* next step, setup openldap and switch

#### Add user group to nginx user
```
pw user show -a  # shows all users
```
