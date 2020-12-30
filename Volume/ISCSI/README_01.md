# ISCSI

Prerequisite: README_00.md

## 
```

```


# iscsiadm -m discovery -t sendtargets -p 192.168.1.31
192.168.1.31:3260,1 iqn.2020-12.com.example.iscsi-server:disk01
# iscsiadm -m discovery -t sendtargets -p 192.168.1.32
192.168.1.32:3260,1 iqn.2020-12.com.example.iscsi-server:disk01




# Reference
https://github.com/kubernetes/examples/tree/master/volumes/iscsi
https://kubernetes.io/docs/concepts/storage/volumes/
