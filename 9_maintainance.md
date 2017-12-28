### FreeBSD maintainance

#### Ports

```
# portsnap fetch extract
```
Download and initialize, only for the first time


```
# portsnap fetch update
```
Download and update


```
portmaster -L --index-only| egrep '(ew|ort) version|total install'
```
installed applications compared to the version in the ports collection


```
#!/bin/sh
/usr/sbin/portsnap fetch update && \
/usr/local/sbin/portmaster -L --index-only | egrep '(ew|ort) version|total install'
```
small shell script for checking of new versions


#### portmaster
```
-n  run through all steps, but do not make or install any ports
-a  check all ports, update as necessary
-l  list all installed ports by category
-f  always	rebuild	ports (overrides -i)
-v  verbose output
```

#### PKG
```
pkg version -vIL=
pkg info -qao
```


#### dfd

https://www.freebsd.org/releases/11.1R/installation.html
```
# freebsd-update fetch
# freebsd-update install
# freebsd-update upgrade -r 11.1-RELEASE
# freebsd-update install
```

```
root@wallace:/home/hoppel # freebsd-update upgrade -r 11.1-RELEASE
src component not installed, skipped
Looking up update.FreeBSD.org mirrors... 3 mirrors found.
Fetching metadata signature for 11.0-RELEASE from update5.freebsd.org... done.
Fetching metadata index... done.
Fetching 1 metadata files... done.
Inspecting system... done.

The following components of FreeBSD seem to be installed:
kernel/generic world/base

The following components of FreeBSD do not seem to be installed:
kernel/generic-dbg world/base-dbg world/doc world/lib32 world/lib32-dbg

Does this look reasonable (y/n)? y

Fetching metadata signature for 11.1-RELEASE from update5.freebsd.org... done.
Fetching metadata index... done.
Fetching 1 metadata patches. done.
Applying metadata patches... done.
Fetching 1 metadata files... done.
Inspecting system... done.
```

```
Attempting to automatically merge changes in files... done.

The following changes, which occurred between FreeBSD 11.0-RELEASE and
FreeBSD 11.1-RELEASE have been merged into /etc/ssh/sshd_config:
--- current version
+++ new version
@@ -1,7 +1,7 @@
 #	$OpenBSD: sshd_config,v 1.98 2016/02/17 05:29:04 djm Exp $
-#	$FreeBSD: releng/11.0/crypto/openssh/sshd_config 296633 2016-03-11 00:15:29Z des $
+#	$FreeBSD: releng/11.1/crypto/openssh/sshd_config 311915 2017-01-11 05:56:40Z delphij $
 
 # This is the sshd server system-wide configuration file.  See
 # sshd_config(5) for more information.
 
 # This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin
@@ -135,11 +135,12 @@
 #UseDNS yes
 #PidFile /var/run/sshd.pid
 #MaxStartups 10:30:100
 #PermitTunnel no
 #ChrootDirectory none
-#VersionAddendum FreeBSD-20160310
+#UseBlacklist no
+#VersionAddendum FreeBSD-20161230
 
 # no default banner path
 #Banner none
 
 # override default of no subsystems
Does this look reasonable (y/n)? 
``