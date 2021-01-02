
## Create ISCSI server(target) on Ubuntu 20.04

The ISCSI server(target)

```
# apt update
# apt install -y tgt
# mkdir -p /var/iscsi/blocks
# dd if=/dev/zero of=/var/iscsi/blocks/disk01.img bs=1M count=1024 status=progress
```

* /etc/tgt/conf.d/host01.localdomain.disk01.conf
```
<target iqn.2020-12.com.example.dnuc01:disk01>
  backing-store /var/iscsi/blocks/disk01.img
  initiator-name iqn.2020-12.com.example.dnuc01:initiator01
  incominguser initiator secret
</target>
```

If you want to publish a device or partition (not sparse file), you can specify it at the section `backing-store` like below.

* /etc/tgt/conf.d/host01.localdomain.disk01.conf
```
<target iqn.2020-12.com.example.dnuc01:sdb>
  backing-store /dev/sdb
  initiator-name iqn.2020-12.com.example.foo:initiator01
  incominguser initiator secret
</target>
```

// Initiator name should be `iqn.YYYY-MM.<reverse-domain-name>:<initiator-name>`

Restart tgt.

```
# systemctl restart tgt
```

## Install server(target) on Ubuntu 20.04

```
# apt update
# apt install -y open-iscsi
# systemctl enable iscsid
```

* /etc/iscsi/initiatorname.iscsi
```
InitiatorName=iqn.2020-12.com.example.dnuc01:initiator01
```

* /etc/iscsi/iscsid.conf (diff)
```
@@ -38,10 +38,10 @@
 #*****************

 # To request that the iscsi initd scripts startup a session set to "automatic".
-# node.startup = automatic
+node.startup = automatic
 #
 # To manually startup the session set to "manual". The default is manual.
-node.startup = manual
+# node.startup = manual

 # For "automatic" startup nodes, setting this to "Yes" will try logins on each
 # available iface until one succeeds, and then stop.  The default "No" will try
@@ -58,8 +58,8 @@

 # To set a CHAP username and password for initiator
 # authentication by the target(s), uncomment the following lines:
-#node.session.auth.username = username
-#node.session.auth.password = password
+node.session.auth.username = initiator
+node.session.auth.password = secret

 # To set a CHAP username and password for target(s)
 # authentication by the initiator, uncomment the following lines:
```

You can specify not to use multipath if you don't want to use it before you login.

* /etc/multipath.conf
```
defaults {
    user_friendly_names yes
}
blacklist {
    devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st|sd[a-z])[0-9]*"
}
```

Then restart iscsid.

```
# systemctl restart iscsid
```

Discover devices on the targets and login to them.

```
# iscsiadm -m discovery -t sendtargets -p 192.168.1.31
192.168.1.31:3260,1 iqn.2020-12.com.example.dnuc01:disk01

# iscsiadm -m node -p 192.168.1.31 -T iqn.2020-12.com.example.dnuc01:disk01 --login
Logging in to [iface: default, target: iqn.2020-12.com.example.dnuc01:disk01, portal: 192.168.1.31,3260] (multiple)
Login to [iface: default, target: iqn.2020-12.com.example.dnuc01:disk01, portal: 192.168.1.31,3260] successful.

# # You can also login to all the available targets with the following command
# iscsiadm -m node -p 192.168.1.31 --login
...
```

Now you can see the device on you client.
You can do anything like simple raw device

```
# dmesg | grep -P '\[sd[a-z]\]'
[1524801.348474] sd 3:0:0:1: [sda] 2097152 512-byte logical blocks: (1.07 GB/1.00 GiB)
[1524801.348476] sd 3:0:0:1: [sda] 4096-byte physical blocks
[1524801.348579] sd 3:0:0:1: [sda] Write Protect is off
[1524801.348580] sd 3:0:0:1: [sda] Mode Sense: 69 00 10 08
[1524801.348793] sd 3:0:0:1: [sda] Write cache: enabled, read cache: enabled, supports DPO and FUA
[1524801.393868] sd 3:0:0:1: [sda] Attached SCSI disk
```

```
# iscsiadm -m node -p 192.168.1.31 -T iqn.2020-12.com.example.dnuc01:disk01 --login
```

If you want to logout

```
# iscsiadm -m node -p 192.168.1.31 -T iqn.2020-12.com.example.dnuc01:disk01 --logout
```

## Specify same initiator that located in other hosts

You can specify the initiator that located between other hosts.
It will be shown from client as a mpath device.
First, create targets that have same name of the initiator like blow.

* /etc/tgt/conf.d/iqn.2020-12.com.example.iscsi-server.conf (on node01 and node02 (ISCSI servers))
```
<target iqn.2020-12.com.example.iscsi-server:disk01>
  backing-store /var/iscsi/blocks/disk01.img
  initiator-name iqn.2020-12.com.example.iscsi-server:iscsi-server
  incominguser initiator secret
</target>
```

* /etc/iscsi/initiatorname.iscsi
```
InitiatorName=iqn.2020-12.com.example.iscsi-server:iscsi-server
```

```
client ~# iscsiadm -m discovery -t sendtargets -p 192.168.1.31
client ~# iscsiadm -m discovery -t sendtargets -p 192.168.1.32
client ~# iscsiadm -m discovery -t sendtargets -p 192.168.1.33
client ~# iscsiadm -m node -p 192.168.1.31 -T iqn.2020-12.com.example.iscsi-server:disk01 --login
client ~# iscsiadm -m node -p 192.168.1.32 -T iqn.2020-12.com.example.iscsi-server:disk01 --login
client ~# iscsiadm ...
client ~# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
...
sda                         8:0    0     1G  0 disk
sdb                         8:16   0     1G  0 disk
...
```

## View of mapping

```
client ~# iscsiadm -m session -P 3 | grep 'Target\|disk'
Target: iqn.2020-12.com.example.dnuc01:dnuc01 (non-flash)
                Target Reset Timeout: 30
                        Attached scsi disk sda          State: running
Target: iqn.2020-12.com.example.iscsi-server:iscsi-server (non-flash)
                Target Reset Timeout: 30
                        Attached scsi disk sdb          State: running
                Target Reset Timeout: 30
                        Attached scsi disk sde          State: running
Target: iqn.2020-12.com.example.iscsi-server:dnuc02 (non-flash)
                Target Reset Timeout: 30
                        Attached scsi disk sdc          State: running
Target: iqn.2020-12.com.example.iscsi-server:dnuc03 (non-flash)
                Target Reset Timeout: 30
                        Attached scsi disk sdd          State: running
```

# Reference
https://linuxhint.com/iscsi_storage_server_ubuntu/
https://askubuntu.com/a/1269917
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/dm_multipath/mpio_configfile
https://github.com/kubernetes/examples/tree/master/volumes/iscsi
https://kubernetes.io/docs/concepts/storage/volumes/
