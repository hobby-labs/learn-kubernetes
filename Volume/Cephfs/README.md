# Servers

dnuc01 (Ceph Deploy, Ceph Monitor, Object Storage)
dnuc02 (Object Storage)
dnuc03 (Object Storage)

## Prerequisite

Ceph Deploy(dnuc01) can access over SSH. Prefer to login without password.

## Build Cephadm

```
dnuc01 ~# curl --silent --remote-name --location https://github.com/ceph/ceph/raw/octopus/src/cephadm/cephadm
dnuc01 ~# chmod +x cephadm
dnuc01 ~# mv cephadm  /usr/local/bin/
dnuc01 ~# cephadm --help
```

## Prepare hosts

```
cat << EOF > /etc/hosts
127.0.0.1 localhost
127.0.1.1 $(uname -n)

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.1.31 dnuc01
192.168.1.32 dnuc02
192.168.1.33 dnuc03
EOF

```

## Install docker on each node

```
{
apt-get update
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
}
```

## Deploy ceph

```
dnuc01 ~# mkdir /etc/ceph
dnuc01 ~# # cephadm bootstrap --mon-ip dnuc01 --initial-dashboard-user admin --initial-dashboard-password secret
dnuc01 ~# cephadm bootstrap --mon-ip 192.168.1.31 --initial-dashboard-user admin --initial-dashboard-password secret
```

Open your browser and see the URL.

```
https://192.168.1.31:8443
```

You can see a login window.

## Install Ceph tools

```
dnuc01 ~# cephadm add-repo --release octopus
dnuc01 ~# cephadm install ceph-common

dnuc01 ~# # Label the nodes with mon
dnuc01 ~# ceph orch host label add dnuc01 mon
```

If you have extra monitors you can add them.
Please see the page(https://computingforgeeks.com/install-ceph-storage-cluster-on-ubuntu-linux-servers/).

Check the hosts and processes.

```
dnuc01 ~# ceph orch host ls
HOST    ADDR    LABELS  STATUS
dnuc01  dnuc01  mon

dnuc01 ~# docker ps
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-grafana.dnuc01
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-alertmanager.dnuc01
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-prometheus.dnuc01
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-node-exporter.dnuc01
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-crash.dnuc01
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-mgr.dnuc01.darjzs
> ceph-bffa7f7e-4e8e-11eb-8336-91ab43728af7-mon.dnuc01
```

## Attach new disk

```
nuc0[123] ~# qemu-img create -f qcow2 ubuntu_$(uname -n) /var/kvm/distros/ubuntu_$(uname -n)/vdb.img 5G
nuc0[123] ~# # Add disk image as persistent volumes
nuc0[123] ~# virsh attach-disk ubuntu_$(uname -n) /var/kvm/distros/ubuntu_$(uname -n)/vdb.img --target vdb --persistent
```

https://www.cyberciti.biz/faq/how-to-add-disk-image-to-kvm-virtual-machine-with-virsh-command/

## Deploy Ceph OSDs

```
dnuc01 ~# # Install the cluster's public SSH key in the new OSD nodes.
dnuc01 ~# ssh-copy-id -f -i /etc/ceph/ceph.pub root@dnuc01
dnuc01 ~# ssh-copy-id -f -i /etc/ceph/ceph.pub root@dnuc02
dnuc01 ~# ssh-copy-id -f -i /etc/ceph/ceph.pub root@dnuc03

dnuc01 ~# # Add hosts to cluster
dnuc01 ~# ceph orch host add dnuc01
dnuc01 ~# ceph orch host add dnuc02
dnuc01 ~# ceph orch host add dnuc03

dnuc01 ~# # Give new nodes labels
dnuc01 ~# ceph orch host label add dnuc01 osd
dnuc01 ~# ceph orch host label add dnuc02 osd
dnuc01 ~# ceph orch host label add dnuc03 osd

dnuc01 ~# # View all devices on storage nodes
dnuc01 ~# ceph orch device ls
Hostname  Path      Type  Serial  Size   Health   Ident  Fault  Available
dnuc01    /dev/vdb  hdd           6442M  Unknown  N/A    N/A    Yes
dnuc02    /dev/vdb  hdd           6442M  Unknown  N/A    N/A    Yes
dnuc03    /dev/vdb  hdd           6442M  Unknown  N/A    N/A    Yes
```

Please check list below if you can not see the devices.

```
> List the storages thet considered available if all of the following conditions are met.
> * The device must have no partitions.
> * The device must not have any LVM state.
> * The device must not be mounted.
> * The device must not contain a file system.
> * The device must not contain a Ceph BlueStore OSD.
> * The device must be larger than 5 GB.
```

```
dnuc01 ~# # Tell Ceph to consume any available and unused storage device
dnuc01 ~# ceph orch daemon add osd dnuc01:/dev/vdb
dnuc01 ~# ceph orch daemon add osd dnuc02:/dev/vdb
dnuc01 ~# ceph orch daemon add osd dnuc03:/dev/vdb
dnuc01 ~# # Check ceph status
dnuc01 ~# ceph -s
  cluster:
    id:     bffa7f7e-4e8e-11eb-8336-91ab43728af7
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum dnuc01,dnuc03,dnuc02 (age 13m)
    mgr: dnuc01.darjzs(active, since 12m), standbys: dnuc02.xhdxud
    osd: 3 osds: 3 up (since 3m), 3 in (since 3m)

  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 15 GiB / 18 GiB avail  # <- Sum of available volumes that consists of 3 nodes (dnuc01 - dnuc03).
    pgs:     1 active+clean

```

# Use ceph on Kubernetes

## Get kubernetes key

```
ceph auth get-key client.admin
```

# Reference
* Install Ceph 15 (Octopus) Storage Cluster on Ubuntu 20.04
https://computingforgeeks.com/install-ceph-storage-cluster-on-ubuntu-linux-servers/

* Ceph Persistent Storage for Kubernetes with Cephfs
https://computingforgeeks.com/ceph-persistent-storage-for-kubernetes-with-cephfs/

