# ISCSI

Prerequisite: README_00.md

Abstruction.

dnuc01
dnuc02
dnuc03


## 
```
# iscsiadm -m discovery -t sendtargets -p 192.168.1.31
192.168.1.31:3260,1 iqn.2020-12.com.example.iscsi-server:disk01
# iscsiadm -m discovery -t sendtargets -p 192.168.1.32
192.168.1.32:3260,1 iqn.2020-12.com.example.iscsi-server:disk01
```

## Trouble shooting

Pod succeeded in AttachVolume.Attach but failed to mount it after several seconds.

```
default     4m12s       Normal    Scheduled                pod/iscsipd                                    Successfully assigned default/iscsipd to nuc03
default     4m13s       Normal    SuccessfulAttachVolume   pod/iscsipd                                    AttachVolume.Attach succeeded for volume "iscsivol"
default     2s          Warning   FailedMount              pod/iscsipd                                    MountVolume.WaitForAttach failed for volume "iscsivol" : failed to get any path for iscsi disk, last err seen:
Timed out waiting for device at path /dev/disk/by-path/ip-192.168.1.31:3260-iscsi-iqn.2020-12.com.example.dnuc01:dnuc01-lun-0 after 30s
default     2m10s       Warning   FailedMount              pod/iscsipd                                    Unable to attach or mount volumes: unmounted volumes=[iscsivol], unattached volumes=[iscsivol default-token-5p6x2]: timed out waiting for the condition
```

In my case, device path `/dev/disk/by-path/ip-192.168.1.31:3260-iscsi-iqn.2020-12.com.example.dnuc01:dnuc01-lun-0` is not found due to wrong LUN number `0`.
The process I could found it is as below.

* Discover the device and logon to it
```
# iscsiadm -m discovery -t sendtargets -p 192.168.1.31
# iscsiadm -m node -p 192.168.1.31 -T iqn.2020-12.com.example.dnuc01:dnuc01 --login
```

* Check the device path whether the device path that printed in the log is correct
```
# ls -l /dev/disk/by-path/
/dev/disk/by-path/ip-192.168.1.31:3260-iscsi-iqn.2020-12.com.example.dnuc01:dnuc01-lun-1
-> LUN number was 1 not 0.
```

* Fix the LUN number 0 in the manifest to 1
```

```

# Reference
https://github.com/kubernetes/examples/tree/master/volumes/iscsi
https://kubernetes.io/docs/concepts/storage/volumes/
