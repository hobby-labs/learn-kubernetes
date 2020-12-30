# ISCSI

## Install initiator on Ubuntu 20.04


## Create ISCSI client(initiator) on Ubuntu 20.04

The ISCSI Server(initiator)

```
# apt update
# apt install -y tgt
# mkdir -p /var/iscsi/blocks
# dd if=/dev/zero of=/var/iscsi/blocks/disk01.img bs=1M count=1024 status=progress
```

* /etc/tgt/conf.d/host01.localdomain.disk01.conf
```
<target host01.localdomain:disk01>
  backing-store /var/iscsi/blocks/disk01.img
  initiator-name host01.localdomain:initiator01
  incominguser initiator secret
</target>
```

If you want to publish a device or partition (not sparse file), you can specify it at the section `backing-store` like below.

* /etc/tgt/conf.d/host01.localdomain.disk01.conf
```
<target host01.localdomain:disk01>
  backing-store /dev/sdb
  initiator-name host01.localdomain:initiator01
  incominguser initiator secret
</target>
```

Restart tgt.

```
# systemctl restart tgt
```

# Reference
https://linuxhint.com/iscsi_storage_server_ubuntu/
https://github.com/kubernetes/examples/tree/master/volumes/iscsi
https://kubernetes.io/docs/concepts/storage/volumes/
