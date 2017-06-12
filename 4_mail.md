## mail
* http://www.purplehat.org/?page_id=7
* https://blog.iandreev.com/?p=1604
* https://gist.github.com/leifdenby/93b41253056f595d62a0f0b21a6f011d
* https://workaround.org/ispmail/jessie/preparing-database

### database

#### user management

##### with roles
```
CREATE DATABASE mailserver;

CREATE ROLE 'mailserver_read_role';
CREATE ROLE 'mailserver_write_role'

GRANT SELECT ON mailserver.* TO 'mailserver_read_role';
GRANT SELECT, INSERT, UPDATE, DELETE ON mailserver.* TO 'mailserver_write_role';

CREATE USER 'mailserver_ro'@'10.1.1.3' IDENTIFIED BY 'changeMe';
CREATE USER 'mailserver_rw'@'10.1.1.3' IDENTIFIED BY 'changeMe';

GRANT 'mailserver_read_role' TO 'mailserver_ro'@'10.1.1.3', 'mailserver_rw'@'10.1.1.3';
GRANT 'mailserver_write_role' TO 'mailserver_rw'@'10.1.1.3';


SET DEFAULT ROLE ALL TO 'mailserver_ro'@'10.1.1.3', 'mailserver_rw'@'10.1.1.3';
```

##### without roles
```
CREATE DATABASE system_mail_config;
CREATE USER 'system_postfix'@'10.1.1.3' IDENTIFIED BY 'changeMe';
mysql> GRANT SELECT ON system_mail_config.* TO 'system_postfix'@'10.1.1.3';
```



#### create tables
##### Domain
```
### DOMAIN ###
# id          a unique number identifying each row. It is added automatically by the database.
# name        name of the domain you want to receive email for
# description description for the domain
# created     creation date of the row / domain entry (automatically)
# modified    modification date (automatically)
# active      flag to activate or deactivate the domain
CREATE TABLE IF NOT EXISTS `domain` (
    `id`          INT(11)       NOT NULL auto_increment,
	`name`        VARCHAR(255)  NOT NULL,
	`description` VARCHAR(2048) NOT NULL DEFAULT '',
	`created`     DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP, 
	`modified`    DATETIME      NOT NULL ON UPDATE CURRENT_TIMESTAMP, 
	`active`      TINYINT(1)    NOT NULL DEFAULT '1',
	PRIMARY KEY (`id`),
	UNIQUE KEY (`name`) 
)
ENGINE=InnoDB 
DEFAULT CHARSET=utf8;  
```

##### Mailbox / User
```
### MAILBOX ###
# id         unique number identifying each row. It is added automatically 
#            by the database.
# domain_id  Contains the number of the domain’s id in the virtual_domains 
#            table. A “delete cascade” makes sure that if a domain is deleted 
#            that all user accounts in that domain are also deleted to avoid 
#            orphaned rows.
# email      email address of the mail account
# password   The hashed password of the mail account. It is prepended by the 
#            hash algorithm. By default it is {SHA256-CRYPT} encrypted. But 
#            you may have legacy users from former ISPmail installations that 
#            still use {PLAIN-MD5}. Adding the hashing algorithm allows you 
#            to have different kinds of hashes.
CREATE TABLE IF NOT EXISTS `mailbox` (
    `id`          INT(11)       NOT NULL auto_increment,
    `domain_id`   INT(11)       NOT NULL,
    `email`       VARCHAR(255)  NOT NULL,
    `password`    VARCHAR(255)  NOT NULL,
    `maildir`     VARCHAR(255)  NOT NULL,
    `quota`       INT(10)       NOT NULL DEFAUL 0,
	`created`     DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP, 
	`modified`    DATETIME      NOT NULL ON UPDATE CURRENT_TIMESTAMP, 
	`active`      TINYINT(1)    NOT NULL DEFAULT '1',
    PRIMARY KEY (`id`),
    UNIQUE KEY `email` (`email`),
    FOREIGN KEY (domain_id) REFERENCES domain(id) ON DELETE CASCADE
) 
ENGINE=InnoDB 
DEFAULT CHARSET=utf8;
```

##### Alias
```
### ALIAS ###
# id          unique number identifying each row. It is added automatically 
#             by the database.
# domain_id   Contains the number of the domain’s id in the virtual_domains 
#             table. A “delete cascade” makes sure that if a domain is deleted 
#             that all user accounts in that domain are also deleted to avoid 
#             orphaned rows.
# source      email address that the email was actually sent to. In case of 
#             catch-all addresses (that accept any address in a domain) the 
#             source looks like “@example.org”
# destination email address that the email should instead be sent to
CREATE TABLE IF NOT EXISTS `alias` (
    `id`          INT(11)       NOT NULL auto_increment,
    `domain_id`   INT(11)       NOT NULL,
    `source`      VARCHAR(255)  NOT NULL,
    `destination` VARCHAR(255)  NOT NULL,
	`created`     DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP, 
	`modified`    DATETIME      NOT NULL ON UPDATE CURRENT_TIMESTAMP, 
	`active`      TINYINT(1)    NOT NULL DEFAULT '1',
    PRIMARY KEY (`id`),
    FOREIGN KEY (domain_id) REFERENCES domain(id) ON DELETE CASCADE
) 
ENGINE=InnoDB 
DEFAULT CHARSET=utf8;
```

##### Passwort
```
dovecot pw -s SHA256-CRYPT”
```


### postfix

### dovecot2

```
===> Creating groups.
Creating group 'dovecot' with gid '143'.
Creating group 'dovenull' with gid '144'.
===> Creating users
Creating user 'dovecot' with uid '143'.
Creating user 'dovenull' with uid '144'.
---------------------------------------------------------------------

 You must create the configuration files yourself. Copy them over
 to /usr/local/etc/dovecot and edit them as desired:

 	cp -R /usr/local/etc/dovecot/example-config/* \
 		/usr/local/etc/dovecot

 The default configuration includes IMAP and POP3 services, will
 authenticate users agains the system's passwd file, and will use
 the default /var/mail/$USER mbox files.

 Next, enable dovecot in /etc/rc.conf:

 	dovecot_enable="YES"


---------------------------------------------------------------------

To avoid a risk of mailbox corruption, do not enable the
security.bsd.see_other_uids or .see_other_guids sysctls if Dovecot
is storing mail for multiple concurrent users (PR 218392).

---------------------------------------------------------------------

 If you want to be able to search within attachments using the
 decode2text plugin, you'll need to install textproc/catdoc, and
 one of graphics/xpdf or graphics/poppler-utils.

---------------------------------------------------------------------

===> SECURITY REPORT: 
      This port has installed the following files which may act as network
      servers and may therefore pose a remote security risk to the system.
/usr/local/lib/dovecot/libdovecot.so.0.0.0

      This port has installed the following startup scripts which may cause
      these network services to be started at boot time.
/usr/local/etc/rc.d/dovecot

      If there are vulnerabilities in these programs there may be a security
      risk to the system. FreeBSD makes no guarantee about the security of
      ports included in the Ports Collection. Please type 'make deinstall'
      to deinstall the port if this is a concern.

      For more information, and contact details about the security
      status of this software, see the following webpage: 
http://www.dovecot.org/
===>  Cleaning for dovecot2-2.2.30.1
```

### dovecot-pigeonhole

```
[mail] Installing dovecot-pigeonhole-0.4.18_3...
---------------------------------------------------------------------

 This port assumes you are familiar with Dovecot and have it installed
 and running on the system you have installed this plugin on.

 You can enable the plugin with this directive in your dovecot.conf:

     protocol lda {
       # Support for dynamically loadable plugins. mail_plugins is
       # a space separated list of plugins to load.
       mail_plugins = sieve # ... other plugins like quota
     }

 Further information on configuration can be found at:

  http://wiki2.dovecot.org/Pigeonhole

---------------------------------------------------------------------

===>  Cleaning for dovecot-pigeonhole-0.4.18_3

```

### configuration
/etc/rc.conf
```
postfix_enable="YES"
dovecot_enable="YES"
```

postfix
```
cd /usr/local/etc/postfix
cp main.cf main.cf.ORIG
cp master.cf master.cf.ORIG
### main.cf
myhostname = www.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
home_mailbox = Maildir/

### end of main.cf
# Virtual domain config
virtual_mailbox_domains = /usr/local/etc/postfix/virtual_domains
virtual_mailbox_base = /var/mail/vhosts
virtual_mailbox_maps = hash:/usr/local/etc/postfix/vmailbox
# Make sure you replace these UID:GID numbers
virtual_minimum_uid = 1002
virtual_uid_maps = static:1002
virtual_gid_maps = static:1001
virtual_alias_maps = hash:/usr/local/etc/postfix/virtual
```

create new file */usr/local/etc/postfix/virtual_domains*
```
cd /usr/local/etc/postfix/
touch virtual_domains

vi virtual_domains
#  Put each domain in a separate line.
domain-one.com
domain-two.net
domain-three.org
```

*Create the mail directory, sub-directories for the domains and assign the proper permissions. This is where the mail will be stored for all virtual domains.*
```
mkdir /var/mail/vhosts
chgrp -R vpostfix /var/mail
cd /var/mail/vhosts
mkdir domain-one.com
mkdir domain-two.net
mkdir domain-three.org
cd ..
chown -R vpostfix:vpostfix vhosts
```

*Once you do that, postfix will create the “Maildir” directories automatically and assign the proper permissions once an e-mail hits these destinations. Finally, create a file /usr/local/etc/postfix/vmailbox and add all of the users that will receive e-mails.*

*NOTE: Make sure you end up each line with “/”, otherwise mail won’t be delivered.*
```
joe@domain-one.com        domain-one.com/joe/
bill@domain-one.com       domain-one.com/bill/
@domain-one.com           domain-one.com/catch-all/
joe@domain-two.net        domain-two.net/joe/
```

user
```
pw groupadd vpostfix && pw useradd vpostfix -g vpostfix -s /usr/sbin/nologin -c "Virtual Postfix user" -d /var/empty
grep vpostfix /etc/passwd
vpostfix:*:1002:1001:Virtual Postfix user:/var/empty:/usr/sbin/nologin
```




####
spamassassin amavisd-new clamav
* https://gist.github.com/leifdenby/93b41253056f595d62a0f0b21a6f011d
* https://blog.entwicklerseite.de/configuration/server/email/dovecot/