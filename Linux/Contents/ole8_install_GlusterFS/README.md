
# GlusterFS on Oracle Linux


## Table Of Contents

- [GlusterFS Server Side Setup](#glusterfs-server-side-setup)
  * [Introduction](#introduction)
  * [Set Hostname](#set-hostname)
  * [Enable GlusterFS Package](#enable-glusterfs-package)
  * [Install GlusterFS](#install-glusterfs)
  * [Format Disks](#format-disks)
  * [Create Directory](#create-directory)
  * [Add The Mount to FSTAB](#add-the-mount-to-fstab)
  * [Mount the Disk](#mount-the-disk)
  * [Add Hostnames To Hostfile](#add-hostnames-to-hostfile)
  * [Enable/Start GlusterFS Service](#enable-start-glusterfs-service)
  * [Allow GlusterFS on Firewall](#allow-glusterfs-on-firewall)
  * [Create Trusted Storage Pool](#create-trusted-storage-pool)
  * [Show Peer Status](#show-peer-status)
  * [List Storage Pool](#list-storage-pool)
  * [Create Volume](#create-volume)
  * [Start Volume](#start-volume)
  * [Show Volume Info](#show-volume-info)
- [Client Side Setup](#client-side-setup)
  * [Enable GlusterFS Package](#enable-glusterfs-package-1)
  * [Install GlusterFS Client](#install-glusterfs-client)
  * [Create Directory](#create-directory-1)
  * [Add Hostnames To Hostfile](#add-hostnames-to-hostfile-1)
  * [Mount GlusterFS Volume](#mount-glusterfs-volume)
  * [Set Permissions](#set-permissions)
  * [Add The Mount to FSTAB](#add-the-mount-to-fstab-1)
  * [Touch File](#touch-file)
  * [Check GlusterFS Nodes](#check-glusterfs-nodes)
- [Delete Volume](#delete-volume)
  * [Check Volume Mount Point on Clients](#check-volume-mount-point-on-clients)
  * [Umount Volume On all Clients](#umount-volume-on-all-clients)
  * [Stop Volume](#stop-volume)
  * [Delete Volume](#delete-volume-1)
  * [Confirm Volume Deletion](#confirm-volume-deletion)


## GlusterFS Server Side Setup


### Introduction 

GlusterFS (Gluster File System) is a free, open-source distributed file system that provides scalable and highly available storage for large-scale data processing, virtualization, and cloud infrastructure. It is designed to allow multiple servers to work together to provide a unified and consistent view of a file system, enabling applications to access data as if it were stored on a single server.

GlusterFS is based on a client-server architecture, where the storage servers provide storage capacity and the clients access and manage the data. It uses a distributed hash table (DHT) to index and store files across multiple storage servers, which enables it to scale to support very large file systems with petabytes of data.

One of the key benefits of GlusterFS is its ability to automatically distribute data across multiple storage servers, ensuring high availability and redundancy. It also provides features such as data replication, data tiering, and snapshots, making it well-suited for applications that require high-performance and reliable storage.

Check the system Requirement by access this [link](https://docs.oracle.com/en/operating-systems/oracle-linux/gluster-storage/gluster-install-upgrade.html#2.2-Operating-System-Requirements)

![](./attachments/Pasted%20image%2020240323020733.png)


Let's get started, and proceed to the installation steps: 

### Set Hostname 


you can set on your own a server name based on your choice:- 

```bash
# Set hostname for each host
hostnamectl set-hostname ol8-node1
hostnamectl set-hostname ol8-node2
hostnamectl set-hostname ol8-nodex
```


### Enable GlusterFS Package 


Apply below step on all the nodes that you need to install and configure the GlusterFS for them:-

```bash
# Enable access to the Gluster packages.
sudo dnf install oracle-gluster-release-el8 -y
sudo dnf config-manager --enable ol8_gluster_appstream ol8_baseos_latest ol8_appstream
```

### Install GlusterFS Online


Install GlusterFS server on all servers or nodes that you need GusterFS to operate on top of them:- 

```bash
# Install the Gluster server packages.
sudo dnf install @glusterfs/server -y
```

###  Install GlusterFS Offline

TODO

### Format Disks

Mount the disks on the nodes each node has separate Disk drive, and apply below commands on all nodes:- 

> CAUTION: before do below steps need to make sure that you are working on right disk below action will erase any exist data on the disk.

```bash
# Create an XFS file system for the cluster data.
# The following commands create an XFS file system on the /dev/sdb device

## create LVM
sudo pvcreate /dev/sdb
sudo vgcreate gfsvg /dev/sd
sudo lvcreate gfsvg -n u02 -l 100%FREE
sudo lvs 
sudo vgs

# create file system to logical volume 
sudo mkfs.xfs -f -i size=512 -L glusterfs /dev/mapper/gfsvg-u02

```

### Create Directory

```bash
# create gluster data Direcotry
sudo mkdir -p /data/gv0
```


### Add The Mount to FSTAB

```bash
# After creating the XFS file system on the /dev/sdb device, which is then mounted on the directory /data/gv0. An entry is added to /etc/fstab to make the mount permanent.

# if you using -L with mkfs you can user this 
sudo echo 'LABEL=glusterfs /data/gv0 xfs defaults 0 0'|sudo tee -a /etc/fstab

# if you are not labeling 
sudo echo '/dev/mapper/gfsvg-u02 /data/gv0 xfs defaults 0 0'|sudo tee -a /etc/fstab
```

### Mount the Disk

```bash
# After we added the disk in fstab we can test this be running below command should work with no error if all prevouse steps are okay.
sudo mount -a
```

### Add Hostnames To Hostfile

If you don't have DNS, you can add all nodes hostnames to each node host file as shown below:

```bash
# Ensure that each node has a resolvable name across all the nodes in the pool.
# If your site is not set up with DNS resolution for your host names, you can use the /etc/hosts file instead. 
# You would edit the file of each node to contain entries for all of your Gluster nodes.

sudo vim /etc/hosts

# add 
10.10.20.27     ol8-node1
10.10.20.28     ol8-node2
10.10.20.29     ol8-node3
```

### Enable/Start GlusterFS Service

Run below command on all glusterfs nodes:

```bash
# Enable/Start the Gluster server service.
sudo systemctl enable --now glusterd
```

### Allow GlusterFS on Firewall

```bash
#Configure the firewall to allow traffic on the ports that are specifically used by Gluster.
sudo firewall-cmd --permanent --add-service=glusterfs
sudo firewall-cmd --reload
```


### Create Trusted Storage Pool

Run Below command from first node only:- 

```bash
# Create the trusted storage pool
# from node1 run below commands 
sudo gluster peer probe ol8-node2
sudo gluster peer probe ol8-node3
```

### Show Peer Status

Note: you need root or sudo privilege.

```bash
# To list the peer members of the pool, type the following:
sudo gluster peer status
```


### List Storage Pool

Note: you need root or sudo privilege, and run it from all nodes

```bash
# from first,second and third node run below command to list pool 
sudo gluster pool list
```


### Create Volume 


```bash
# Create the volume out of the nodes in the pool. 

# create dir on all nodes called brick inside the volume. 
sudo mkdir /data/gv0/brick

# replica Volume
sudo gluster volume create DataVol replica 3 ol8-node{1,2,3}:/data/gv0/brick

# or Distributed Volume
sudo gluster volume create DataVol ol8-node{1,2,3}:/data/gv0/brick

```

### Start Volume

```bash
# Start the volume.
sudo gluster volume start DataVol
```


### Show Volume Info 

```bash
# To display information about the volume, type:
sudo gluster volume info
```


## Client Side Setup

Client is the Operating System that will use the volume as  mounted  GlusterFS disk.

### Enable GlusterFS Package 

```bash
# Enable access to the Gluster packages.
sudo dnf install oracle-gluster-release-el8 -y
sudo dnf config-manager --enable ol8_gluster_appstream ol8_baseos_latest ol8_appstream
sudo dnf module enable glusterfs -y
```

### Install GlusterFS Client

```bash
# Install the Gluster client packages.
sudo dnf install @glusterfs/client -y
```

### Create Directory 

```bash
# Create the directory for the client’s data.

sudo mkdir /mydata
```

### Add Hostnames To Hostfile

If you don't have DNS, you can add all nodes hostname to the client host file as shown below:

```bash
# Ensure that each node has a resolvable name across all of clients.

sudo vim /etc/hosts

# add 
10.10.20.27     ol8-node1
10.10.20.28     ol8-node2
10.10.20.29     ol8-node3
```

### Mount GlusterFS Volume

```bash
# Mount the Gluster volume on the directory
sudo mount -t glusterfs ol8-node1:DataVol /mydata
```

### Set Permissions 

```bash
# Set appropriate permissions to the new mount, for example:
sudo chmod 777 /mydata
```

### Add The Mount to FSTAB

```bash
# To make the mount permanent, add it to the /etc/fstab, for example, by typing the following command:
sudo echo 'ol8-node01:DataVol /mydata glusterfs defaults,_netdev 0 0'|sudo tee -a /etc/fstab

# if you are using label use below command 
sudo echo 'LABEL=ol8-node01:DataVol /mydata glusterfs defaults,_netdev 0 0'|sudo tee -a /etc/fstab
```


### Touch File 

```bash
touch /mydata/testfile
# or 
echo "test data from client" > /mydata/testfile
```

### Check GlusterFS Nodes

See if the GlusterFS replicating the file or not. 

```bash
# On each of the Gluster server node, run the following command:
ls -l /data/gv0/brick
# you should be able to see the file on all nodes
```


## Delete Volume 

>Note: Deleting a volume will permanently delete all the data stored in that volume. So, make sure to take a backup of the data before deleting the volume, if needed.

### Check Volume Mount Point on Clients 

First, make sure that the volume is not being used by any client and is not mounted anywhere. You can check this by running the following commands:

```bash
# on client side umount volumes mount point 
df -h | grep DataVol
ol8-node1:DataVol    20G  380M   20G   2% /mydata

# This will show you the status of all the volumes. Make sure that the volume you want to delete is not being used or mounted anywhere.
gluster volume status

```

### Umount Volume On all Clients

```bash
# umount this volume on all clients that use this volume
umount /mydata
```


### Stop Volume

Once you have confirmed that the volume is not being used or mounted, you can stop the volume by running the following command:

```bash
gluster volume stop DataVol
```

### Delete Volume

After stopping the volume, you can delete it by running the following command:

```bash
# This will delete the volume and its configuration from the GlusterFS cluster.
gluster volume delete DataVol
```

### Confirm Volume Deletion

Finally, you can confirm that the volume has been deleted by running the following command:

```bash
# This should not show the deleted volume in the list.
gluster volume list
```

