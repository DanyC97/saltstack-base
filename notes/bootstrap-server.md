
### CentOS 7 Network Install

#### Load the OS and configure Ethernet interface

1. Add MAC and IP address to /etc/dhcp/dhcpd.conf
1. Boot server from USB stick [https://github.com/dkilcy/saltstack-base/notes/kickstart]
2. At GRUB menu, highlight **Install CentOS 7** then hit **Tab**
3. At prompt, remove `quiet` and append `ks=http://10.0.0.6/base.ks ksdev=enp0s20f0` 
4. Wait for install to finish
5. SSH to server from workstation as root to test connectivity
```
[root@workstation2 ~]# ssh root@compute2
The authenticity of host 'compute2 (10.0.0.32)' can't be established.
ECDSA key fingerprint is 75:75:75:26:e8:23:2e:a4:29:21:70:b3:c2:57:9c:91.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'compute2,10.0.0.32' (ECDSA) to the list of known hosts.
root@compute2's password: 
Last login: Sun Jan 18 19:38:29 2015
[root@compute2 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s20f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9000 qdisc mq state UP qlen 1000
    link/ether 0c:c4:7a:31:60:0c brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.32/24 brd 10.0.0.255 scope global dynamic enp0s20f0
       valid_lft 13963sec preferred_lft 13963sec
3: enp0s20f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN qlen 1000
    link/ether 0c:c4:7a:31:60:0d brd ff:ff:ff:ff:ff:ff
4: enp0s20f2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 0c:c4:7a:31:60:0e brd ff:ff:ff:ff:ff:ff
5: enp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 0c:c4:7a:31:60:0f brd ff:ff:ff:ff:ff:ff
[root@compute2 ~]# 
```
6. Add server to Salt master
```
[root@workstation2 ~]# salt-key -L
Accepted Keys:
compute1
...
ring-a6
Unaccepted Keys:
compute2
Rejected Keys:
[root@workstation2 ~]# salt-key -A 
The following keys are going to be accepted:
Unaccepted Keys:
compute2
Proceed? [n/Y] Y
Key for minion compute2 accepted.
[root@workstation2 ~]# 
```
7. Setup Teaming  (TODO: need Salt state)
```
[root@workstation2 tmp]# ls -l
total 24
-rw-r--r--. 1 root root 105 Jan 19 09:00 ifcfg-enp0s20f0
-rw-r--r--. 1 root root 105 Jan 19 09:00 ifcfg-enp0s20f1
-rw-r--r--. 1 root root 105 Jan 19 09:00 ifcfg-enp0s20f2
-rw-r--r--. 1 root root 105 Jan 19 09:00 ifcfg-enp0s20f3
-rw-r--r--. 1 root root 150 Jan 19 09:00 ifcfg-team0
-rw-r--r--. 1 root root 150 Jan 19 09:00 ifcfg-team1
[root@workstation2 tmp]# vi ifcfg-team0
[root@workstation2 tmp]# vi ifcfg-team1
[root@workstation2 tmp]# ls
ifcfg-enp0s20f0  ifcfg-enp0s20f1  ifcfg-enp0s20f2  ifcfg-enp0s20f3  ifcfg-team0  ifcfg-team1
[root@workstation2 tmp]# 
[root@workstation2 tmp]# ls
ifcfg-enp0s20f0  ifcfg-enp0s20f1  ifcfg-enp0s20f2  ifcfg-enp0s20f3  ifcfg-team0  ifcfg-team1
[root@workstation2 tmp]# salt-cp 'compute2' * /etc/sysconfig/network-scripts/
{'compute2': {'/etc/sysconfig/network-scripts/ifcfg-enp0s20f0': True,
              '/etc/sysconfig/network-scripts/ifcfg-enp0s20f1': True,
              '/etc/sysconfig/network-scripts/ifcfg-enp0s20f2': True,
              '/etc/sysconfig/network-scripts/ifcfg-enp0s20f3': True,
              '/etc/sysconfig/network-scripts/ifcfg-team0': True,
              '/etc/sysconfig/network-scripts/ifcfg-team1': True}}
[root@workstation2 tmp]# ssh compute2
The authenticity of host 'compute2 (10.0.0.32)' can't be established.
ECDSA key fingerprint is 75:75:75:26:e8:23:2e:a4:29:21:70:b3:c2:57:9c:91.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'compute2,10.0.0.32' (ECDSA) to the list of known hosts.
root@compute2's password: 
Last login: Mon Jan 19 05:42:15 2015 from workstation2.mgmt
[root@compute2 ~]# systemctl stop NetworkManager
[root@compute2 ~]# systemctl stop NetworkManager.service
[root@compute2 ~]# systemctl disable NetworkManager.service
rm '/etc/systemd/system/multi-user.target.wants/NetworkManager.service'
rm '/etc/systemd/system/dbus-org.freedesktop.NetworkManager.service'
rm '/etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service'
[root@compute2 ~]# service network restart
Restarting network (via systemctl):                        [  OK  ]
[root@compute2 ~]# 
```

