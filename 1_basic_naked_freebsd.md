# Basic naked FreeBSD 11 on a hetzner host

## Hetzner console
* go to robot hetzner console
* activate the rescue system
* give a reboot

## Basic Installation

### 1 Login and Start
* use the root passwd from the hetzner console

```
[root@rescue ~]# bsdinstallimage 
```


### 2 FreeBSD version
```
11.0  FreeBSD 11.0
64  bit
Continue with de keymap
```


### 3 Set Hostname
```
[your-hostname]
```

### 4 Distribution Select
```
[*]  ports
```
I need only the ports. I want a real naked freebsd.


### 5 Partitioning
```
Auto (ZFS)  Guided Root-on-ZFS

T  Pool Type/Disks:  mirror: 2 disks
N  Pool Name         zroot
4  Force 4K Sectors? YES
E  Encrypt Disks?    NO
P  Partition Scheme  GPT (BIOS)
S  Swap Size         4g
M  Mirror Swap?      NO
W  Encrypt Swap?     YES
```
I use software mirror. The swap should be encrypted for security reasons. I have to disable the kernel dev dump option.



### 6 IP Configuration

#### IPv4
Set the DHCP Option to *OFF*. It's a server and u're IP address 'll be shown in the hetzner robot management panel.
* set up the IPv4 settings with the right IP address

#### IPv6
```
ipv6_default_interface="re0"
ifconfig_re0_ipv6="[copy the address from the hetzner console]::1:1/64"
# set a static local interface-route
ipv6_defaultrouter="fe80::1%re0"
```
disable SLAAC for IPv6 configuration. U'll find the right address in u're robot admin panel.

#### IPv4 DNS
IPv4-Adressen unserer rekursiven DNS-Server lauten:
```
213.133.98.98
213.133.99.99
213.133.100.100
```

#### IPv6 DNS
Die IPv6-Adressen unserer rekursiven DNS-Server lauten:
```
2a01:4f8:0:a0a1::add:1010
2a01:4f8:0:a102::add:9999
2a01:4f8:0:a111::add:9898
```


# 7 System Configuration
```
[*]  sshd     Secure shell daemon
[*]  powerd   Adjust CPU frequency dynamically if supported
```

# 8 Add user
* add privilig user
* additional group => wheel
* reboot

# 9 Reboot and Change sshd_conf
* change /etc/ssh/sshd_conf
  Perm no
* restart the sshd deamon

