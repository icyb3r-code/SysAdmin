# Servers Setup

## change hostname

```bash
hostnamectl set-hostname rancher01
```
## remove swap from all nodes

```bash
swapoff -a
sed -i '/swap/d' /etc/fstab
```
## disable firewall

```bash
systemctl disable --now firewalld
```
## disable SELinux

```bash
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
getenforce
```
## disable NetworkManager

If you have not yet installed RKE2, a simple systemctl reload NetworkManager will suffice to install the configuration. If performing this configuration change on a system that already has RKE2 installed, a reboot of the node is necessary to effectively apply the changes.

```bash
vim /etc/NetworkManager/conf.d/rke2-canal.conf
[keyfile]
unmanaged-devices=interface-name:flannel*;interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
```

## Format the LH Disks on all nodes

```bash

[root@rke2-wk01 ~]# 
  Physical volume "/dev/sdb" successfully created.

[root@rke2-wk01 ~]# vgcreate longhorn-vg /dev/sdb
  Volume group "longhorn-vg" successfully created

[root@rke2-wk01 ~]# lvcreate longhorn-vg -n longhorn-lv -l 100%free
  Logical volume "longhorn-lv" created.

[root@rke2-wk01 ~]# mkfs.ext4 /dev/longhorn-vg/longhorn-lv
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 7863296 4k blocks and 1966080 inodes
Filesystem UUID: b70528c7-8a95-42db-8b35-55a4bc89dd9e
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

[root@rke2-wk01 ~]# mkdir /longhorn-dir


[root@rke2-wk01 ~]# vim /etc/fstab
[root@rke2-wk01 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Wed Dec 25 20:11:29 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/ol-root     /                       xfs     defaults        0 0
UUID=678548da-b0f2-4443-9b9a-8aa8f93254b3 /boot                   xfs     defaults        0 0
/dev/longhorn-vg/longhorn-lv	/longhorn-dir	ext4	defaults	0 0
[root@rke2-wk01 ~]#
```


## References
- [RKE2 Documentation](https://docs.rke2.io/)
- [RKE2 Support Matrix](https://www.suse.com/suse-rke2/support-matrix/all-supported-versions/rke2-v1-30/)
- [NetworkManager Configuration](https://docs.rke2.io/known_issues#networkmanager)
- [Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
- [RKE2 Requirements](https://docs.rke2.io/install/requirements)


# RKE2 Server Installation

```bash
# On rancher-01
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=server sh -
# create config dir/file
mkdir -p /etc/rancher/rke2/
# add data location 
echo "data-dir: /data/rancher/rke2" > /etc/rancher/rke2/config.yaml

mkdir -p /data/rancher/rke2

# start and enable for restarts -
systemctl enable --now rke2-server.service

# symlink all the things - kubectl
ln -s $(find /var/lib/rancher/rke2/data/ -name kubectl) /usr/local/bin/kubectl

# symlink all the things - kubectl
ln -s $(find /data/rancher/rke2/bin/ -name kubectl) /usr/local/bin/kubectl

# add kubectl conf with persistence, as per Duane
echo "export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/usr/local/bin/:/var/lib/rancher/rke2/bin/" >> ~/.bashrc

# with custom data dir 
echo "export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/usr/local/bin/:/data/rancher/rke2/bin/" >> ~/.bashrc

source ~/.bashrc

# check node status
kubectl get node

# get the node-token
cat /var/lib/rancher/rke2/server/node-token
K1053ead0bb94d483aff74548a999da3974bdb05bbe4f514b49087774408c036800::server:33631fa267b33269fd5727ed715124f9

# or 
cat /data/rancher/rke2/server/node-token
K10b28449c8563ba88e6dc7593bbfc26d5b88815dd00d2c3042ae272398c9ff491d::server:16e22b62003c7d5c07f27e43cc98dfa3
```

# RKE2 Agent Installation

```bash
# we can export the rancher-01 IP from the first server.
export RANCHER1_IP=10.10.21.140   # <-- change this!

# we add INSTALL_RKE2_TYPE=agent
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=agent sh -

# create config dir/file
mkdir -p /etc/rancher/rke2/

# change the ip to reflect your rancher-01 ip
cat << EOF >> /etc/rancher/rke2/config.yaml
server: https://$RANCHER1_IP:9345
token: K10b28449c8563ba88e6dc7593bbfc26d5b88815dd00d2c3042ae272398c9ff491d::server:16e22b62003c7d5c07f27e43cc98dfa3
data-dir: /data/rancher/rke2
EOF

# enable and start
systemctl enable --now rke2-agent.service
```

## Defaults

```bash
# nodes
kubectl get nodes
NAME        STATUS   ROLES                       AGE     VERSION
rancher01   Ready    control-plane,etcd,master   18m     v1.33.5+rke2r1
rancher02   Ready    <none>                      4m28s   v1.33.5+rke2r1
rancher03   Ready    <none>                      117s    v1.33.5+rke2r

# namespaces
kubectl get ns
NAME              STATUS   AGE
default           Active   19m
kube-node-lease   Active   19m
kube-public       Active   19m
kube-system       Active   19m

# pods 
get pods -A | cut -d " " -f1-4
NAMESPACE   
kube-system   cloud-controller-manager-rancher01
kube-system   etcd-rancher01
kube-system   helm-install-rke2-canal-hpzcx
kube-system   helm-install-rke2-coredns-4r8kn
kube-system   helm-install-rke2-ingress-nginx-9l5tg
kube-system   helm-install-rke2-metrics-server-5wbqk
kube-system   helm-install-rke2-runtimeclasses-xfq57
kube-system   helm-install-rke2-snapshot-controller-crd-4gx7w
kube-system   helm-install-rke2-snapshot-controller-x9zbd
kube-system   kube-apiserver-rancher01
kube-system   kube-controller-manager-rancher01
kube-system   kube-proxy-rancher01
kube-system   kube-proxy-rancher02
kube-system   kube-proxy-rancher03
kube-system   kube-scheduler-rancher01
kube-system   rke2-canal-722tj
kube-system   rke2-canal-hwgq9
kube-system   rke2-canal-scrwr
kube-system   rke2-coredns-rke2-coredns-6464f98784-s4wr6
kube-system   rke2-coredns-rke2-coredns-6464f98784-z5pbn
kube-system   rke2-coredns-rke2-coredns-autoscaler-67bb49dff-cbdhk
kube-system   rke2-ingress-nginx-controller-5msdn
kube-system   rke2-ingress-nginx-controller-7qf6v
kube-system   rke2-ingress-nginx-controller-vlgqv
kube-system   rke2-metrics-server-75d485c65b-n26rv
kube-system   rke2-snapshot-controller-696989ffdd-ngnzz


# 


```

# Rancher Installation

```bash
# on the server rancher-01
# add helm
curl -#L https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# add needed helm charts
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest --force-update
helm repo add jetstack https://charts.jetstack.io --force-update

# still on rancher-01

# helm install jetstack
helm upgrade -i cert-manager jetstack/cert-manager -n cert-manager --create-namespace --set crds.enabled=true

# helm install rancher
# CHANGE the IP to the one for rancher-01
export RANCHER1_IP=10.10.21.140
helm upgrade -i rancher rancher-latest/rancher --create-namespace --namespace cattle-system --set hostname=rancher.$RANCHER1_IP.sslip.io --set bootstrapPassword=bootStrapAllTheThings --set replicas=1
```

# Longhorn Installation

```bash
# install required packages
sudo yum install iscsi-initiator-utils cryptsetup nfs-utils -y

# install longhorn via helm
helm repo add longhorn https://charts.longhorn.io --force-update
helm upgrade -i longhorn longhorn/longhorn --namespace longhorn-system --create-namespace
```


# MinIO Installation on RKE2 

```bash

helm repo add minio https://charts.min.io/
helm repo update

helm install --namespace minio --create-namespace minio minio/minio \
  --set replicas=2 \
  --set persistence.size=10Gi  # Adjust for production
```



Got it ✅ — stopping an **RKE2 environment** (or any Kubernetes cluster) needs care because you’ve got workloads (pods, PVCs, services) across multiple namespaces. If you just power-off servers, you risk:

* etcd corruption
* pods stuck in crashloop or “unknown” state
* data loss in PVs

Here’s a **best practice procedure** for a small cluster like yours (1 control plane + 2 workers):

---

## 🔒 Best Practice to Stop RKE2 Environment Safely

### 1. **Prepare the Cluster**

* Check cluster health first:

  ```bash
  kubectl get nodes -o wide
  kubectl get pods -A -o wide
  kubectl get pvc -A
  ```
* Make sure etcd is healthy (on the control plane):

  ```bash
  /var/lib/rancher/rke2/agent/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/5/fs/usr/local/bin/etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt \
  --cert=/var/lib/rancher/rke2/server/tls/etcd/client.crt \
  --key=/var/lib/rancher/rke2/server/tls/etcd/client.key endpoint health
  
  etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt \
  --cert=/var/lib/rancher/rke2/server/tls/etcd/client.crt \
  --key=/var/lib/rancher/rke2/server/tls/etcd/client.key endpoint health
  ```

### 2. **Gracefully Drain Worker Nodes**

This moves workloads off the workers (if possible).

```bash
kubectl drain <worker-node-name> --ignore-daemonsets --delete-emptydir-data
```

Do this for both worker nodes.
⚠️ If workloads use **local storage** (e.g., emptyDir, hostPath, Longhorn local replicas), draining may cause eviction issues. Handle carefully.

### 3. **Stop Worker Nodes**

On each **worker** node:

```bash
sudo systemctl stop rke2-agent
```

### 4. **Stop the Control Plane**

On the **control plane node**:

```bash
sudo systemctl stop rke2-server
```

This stops:

* kube-apiserver
* controller-manager
* scheduler
* etcd (embedded in RKE2 unless external)

### 5. **Verify All Services Stopped**

```bash
systemctl status rke2-server
systemctl status rke2-agent
```

All should be inactive.

---

## 🔄 Restarting the Cluster

When you’re ready to start again:

1. Start the control plane:

   ```bash
   sudo systemctl start rke2-server
   ```

   Wait for `kubectl get nodes` to show the control plane as `Ready`.

2. Start worker nodes one by one:

   ```bash
   sudo systemctl start rke2-agent
   ```

3. Verify:

   ```bash
   kubectl get nodes -o wide
   kubectl get pods -A -o wide
   ```

---

## ⚠️ Extra Best Practices

* **Snapshots**: Take an etcd snapshot before shutdown:

  ```bash
  rke2 etcd-snapshot save
  ```
* **Order matters**: Always stop **workers first → control plane last**.
  And start **control plane first → workers last**.
* **Storage caution**: If you use Longhorn, ensure volumes are healthy before shutdown:

  ```bash
  kubectl -n longhorn-system get volumes
  ```

---

👉 Boss, do you want me to also give you a **step-by-step checklist script** (like a Bash playbook) you can run every time to safely shutdown/start your RKE2 cluster? That way it’s automated and consistent.



# TODO:
1- secure the ports, keep the firewall open and open the required ports on firewall on each node
2- check what is the Longhorn and how I can admin 
3- check the minio and how I can admin
4- check helm and how I create repo and a chart
5- check how I can 