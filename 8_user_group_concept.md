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
     
