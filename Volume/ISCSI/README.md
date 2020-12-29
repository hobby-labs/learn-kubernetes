# ISCSI

## Create ISCSI Server(initiator) on Ubuntu 20.04

The ISCSI Server(initiator)

```
# apt update
# apt install -y tgt
# mkdir -p /var/iscsi/blocks
# dd if=/dev/zero of=/var/iscsi/blocks/disk01.img bs=1M count=1024 status=progress
```

```
<target host01.localdomain:disk01>
  backing-store /var/iscsi/blocks/disk01.img
  initiator-name hostname.localdomain:initiator01
  incominguser initiator secret
</target>
```



# Reference
https://linuxhint.com/iscsi_storage_server_ubuntu/

