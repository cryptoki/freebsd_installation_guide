## mail
* http://www.purplehat.org/?page_id=7
* https://blog.iandreev.com/?p=1604
* https://gist.github.com/leifdenby/93b41253056f595d62a0f0b21a6f011d
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