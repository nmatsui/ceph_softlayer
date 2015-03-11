ceph_sl
======

This [Ansible](http://www.ansible.com) playbooks create [Ceph](http://ceph.com/) Cluster on [SoftLayer](http://www.softlayer.com/).

## Description

This Ansible playbook setup Ceph v0.80.7(Firefly) Cluster on SoftLayer baremetal server by using [ceph-deploy](https://github.com/ceph/ceph-deploy).

## Requirement

You have to create a client node and three ceph cluster nodes like below:

### Client Node

* Virtual Server
  * Ubuntu 14.04 64bit
  * Upgrade NIC to 1Gbps
* hostname : **client**

### Ceph Cluster Node

* Baremetal Server
  * Ubuntu 14.04 64bit
  * Upgrade NIC to 1Gbps
  * Add second disk (/dev/sdb)
* hostname : **ceph01**, **ceph02**, **ceph03**


## Usage

### Setup Ceph Cluster

1) SSH to client node

`$ ssh root@<public IP addr of client node>`

2) Install ansible to client node

`root@client:~# apt-get install python-dev python-pip -y`
`root@client:~# pip install ansible`

3) make ssh key on client node

`root@client:~# ssh-keygen -q -t rsa -f ~/.ssh/id_rsa -N ""`

 4) copy public key to all nodes

`root@client:~# ssh-copy-id root@<public IP addr of ceph01 node>`
`root@client:~# ssh-copy-id root@<public IP addr of ceph02 node>`
`root@client:~# ssh-copy-id root@<public IP addr of ceph03 node>`
`root@client:~# ssh-copy-id root@<public IP addr of client node>`

5) fdisk /dev/sdb on ceph01, ceph02 and ceph03 node

You have to make two partitions(/dev/sdb1, /dev/sdb2)

6) configure ansible on client node

* [all] : enumerate private IP addr of all nodes
* [manager] : private IP addr of the client node

```text:servers
[all]
10.WWW.WWW.WWW
10.XXX.XXX.XXX
10.YYY.YYY.YYY
10.ZZZ.ZZZ.ZZZ

[client]
10.WWW.WWW.WWW

[client:vars]
client_node=client
storage_nodes=ceph01 ceph02 ceph03
storage_disks=ceph01:sdb2:/dev/sdb1 ceph02:sdb2:/dev/sdb1 ceph03:sdb2:/dev/sdb1
```

7) run ansible playbook on client node

`root@client:playbook# ansible-playbook -i servers site.yml`

### Mount ceph cluster as NAS using CEPH FS

`root@client:~# ceph auth get-key client.admin > /etc/ceph/admin.key`
`root@client:~# mount -t ceph ceph01:6789:/ /mnt/ -o name=admin,secretfile=/etc/ceph/admin.key`

### Mount ceph cluster as Block Device using RBD

`root@client:~# rbd create <image name> --size <Size(MB)>`
`root@client:~# rbd map <image name>`
`root@client:~# fdisk /dev/rbd1`
`root@client:~# mkfs.ext4 /dev/rbd1`
`root@client:~# mount /dev/rbd1 /mnt`

## License

Copyright 2015 Nobuyuki Matsui <nobuyuki.matsui@gmail.com>

Walfisch is released under the Apache License version2.0. The Apache License version2.0 official full text is published at this [link](http://www.apache.org/licenses/LICENSE-2.0.html).
