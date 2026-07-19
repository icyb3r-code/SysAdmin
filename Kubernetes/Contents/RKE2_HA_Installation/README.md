
# RKE2 HA Setup

## RKE2 OS Prerequisites for RHEL 9.6

Below are the **recommended and production-grade OS prerequisites** for installing **RKE2 on Red Hat Enterprise Linux (RHEL) 9.6**, specifically for a **3-node High Availability Control Plane setup**.
### 1. Operating System Requirements

| Requirement  | Details                                                                     |
| ------------ | --------------------------------------------------------------------------- |
| Supported OS | **RHEL 9.6 (x86_64)**                                                       |
| SELinux      | **Permissive** (Recommended), Enforcing is possible with policy adjustments |
| Kernel       | Default RHEL 9 kernel (≥ 5.14)                                              |
| Packages     | Standard minimal installation recommended                                   |

### 2. Hardware Requirements (per Master Node)

|Component|Minimum|Recommended|
|---|---|---|
|vCPU|4 cores|4–8 cores|
|RAM|8 GB|16 GB for large workloads|
|Disk|40 GB|80–120 GB (SSD Preferred)|
|Network|1 Gbps NIC|10 Gbps for production|
### 3. Disable Swap

Kubernetes **does NOT allow swap**.

```
sudo swapoff -a 
sudo sed -i '/swap/d' /etc/fstab
```

### 4. Required Firewall Ports (Master Nodes)

Inbound to Control Plane Nodes

| Port        | Protocol | Purpose               |
| ----------- | -------- | --------------------- |
| 9345        | TCP      | RKE2 server API       |
| 6443        | TCP      | Kubernetes API Server |
| 8472        | UDP      | Canal/VXLAN           |
| 10250       | TCP      | Kubelet               |
| 2379–2380   | TCP      | etcd                  |
| 30000–32767 | TCP      | NodePort Services     |
If using **Calico**, additional BGP ports may apply.

Allow the ports:

```
sudo firewall-cmd --add-port=6443/tcp --permanent
sudo firewall-cmd --add-port=9345/tcp --permanent
sudo firewall-cmd --add-port=8472/udp --permanent
sudo firewall-cmd --add-port=10250/tcp --permanent
sudo firewall-cmd --add-port=2379-2380/tcp --permanent
sudo firewall-cmd --add-port=30000-32767/tcp --permanent
sudo firewall-cmd --reload
```

below are the OS Prerequisites:

### 5. SELinux Configuration

RKE2 supports SELinux, but you need **rke2-selinux** policies.

Install them BEFORE RKE2:

```
sudo dnf install -y container-selinux
sudo rpm -ivh https://rpm.rancher.io/rke2-selinux/rke2-selinux-latest.el9.noarch.rpm
```

If you face issues, set SELinux to permissive:

```
sudo setenforce 0
```

### 6. Required Kernel Modules

Ensure these modules are loaded:

```
sudo modprobe br_netfilter
sudo modprobe overlay
```

Persist them:

```
cat <<EOF | sudo tee /etc/modules-load.d/rke2.conf
br_netfilter
overlay
EOF
```

### 7. Required sysctl Settings


```
cat <<EOF | sudo tee /etc/sysctl.d/90-rke2.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.conf.all.rp_filter = 0
EOF

sudo sysctl --system
```

### 8. Host Name 

```
hostnamectl set-hostname rke2-m01
hostnamectl set-hostname rke2-m02
hostnamectl set-hostname rke2-m03
hostnamectl set-hostname rke2-lb01
hostnamectl set-hostname rke2-w01
```


### 9. HA Load Balancer Requirement

For a **3-node HA control plane**, you MUST have:

- An **external load balancer** (e.g., HAProxy or Nginx)
- One **Virtual IP** (VIP):

Configure your LB to forward traffic to:

- port **6443** (Kube API)
- optionally port **9345** (RKE2 supervisor API)
    

### 10. Disable Firewalld (Optional but Common in RKE2 Setups)

If you prefer using RKE2’s internal handling:

```
sudo systemctl disable --now firewalld
# or 
sudo systemctl stop firewalld
sudo systemctl disable firewalld
# mask the service 
sudo systemctl mask firewalld
```

### 11. Remove Podman / Docker Conflicts

RHEL 9 comes with Podman installed by default, which may conflict with RKE2’s containerd.

```
sudo dnf remove -y podman buildah
```

### 12. Ensure Time Sync (NTP)

Time drift can break etcd.
Install and enable chrony:

```
sudo dnf install -y chrony
sudo systemctl enable --now chronyd
```


### 13. Make Sure RHEL Subscription Is Active

RKE2 requires access to base packages.

Check:

```
subscription-manager status

```

### 14. (Optional) Disable Cloud-Init

If this is a VM template, disable cloud-init:

```
sudo touch /etc/cloud/cloud-init.disabled
```

### 15. Network Requirements

- All master nodes must reach each other on:
    
    - **6443**
    - **9345**
    - **2379–2380**
    - **8472 (UDP)** for VXLAN
        
- DNS must resolve the API endpoint
- Nodes must have **unique** hostnames

### 16. Prepare Directories for RKE2

```
sudo mkdir -p /etc/rancher/rke2
sudo mkdir -p /var/lib/rancher/rke2
```

### 17. Network Manager 

```
sudo cat <<EOF > /etc/NetworkManager/conf.d/rke2-canal.conf
[keyfile]
unmanaged-devices=interface-name:flannel*;interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
EOF
```

### 18. Mount Disks to RKE2

```
mount /etc/rancher/rke2
```


## HAproxy Setup

### Install haproxy 

```
sudo dnf install haproxy
```


### Configure Loadbalancer 

We are going to use here haproxy as L4 loadbalancer

```json

global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon
        
        # SSL configuration for future HTTPS endpoints
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private
        
        # Modern SSL configuration - only secure protocols
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

#---------------------------------------------------------------------
# RKE2 Supervisor/Registration Frontend
# This is where new nodes join the cluster - the secret handshake!
#---------------------------------------------------------------------
frontend rke2_registration_frontend
    bind *:9345
    mode tcp
    option tcplog
    default_backend rke2_registration_backend

#---------------------------------------------------------------------
# RKE2 Supervisor/Registration Backend
# Round-robin between masters for node registration
#---------------------------------------------------------------------
backend rke2_registration_backend
    mode tcp
    balance roundrobin
    option tcp-check
    # Health check ensures we only send traffic to healthy masters
    server cp1 cp1:9345 check
    server cp2 cp2:9345 check
    server cp3 cp3:9345 check

#---------------------------------------------------------------------
# Kubernetes API Frontend
# This is where kubectl commands and apps connect
#---------------------------------------------------------------------
frontend k8s_api_frontend
    bind *:6443
    mode tcp
    option tcplog
    default_backend k8s_api_backend

#---------------------------------------------------------------------
# Kubernetes API Backend
# Distributes API requests across all masters
#---------------------------------------------------------------------
backend k8s_api_backend
    mode tcp
    balance roundrobin
    option tcp-check
    # TCP health checks on the API port
    server cp1 cp1:6443 check
    server cp2 cp2:6443 check
    server cp3 cp3:6443 check

#---------------------------------------------------------------------
# Statistics Page (Optional but useful for monitoring)
#---------------------------------------------------------------------
listen stats
    bind *:8080
    stats enable
    stats uri /stats
    stats refresh 30s
    stats show-node
    stats auth admin:strongpassword  # Change this password!

```

check haproxy configurations 

```
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

create required dirs  & enable start service 

```
sudo mkdir -p /run/haproxy
sudo chown haproxy:haproxy /run/haproxy

systemctl enable --now haproxy
```


## Configure Control Planes 

### Configure RKE2 First Node

we need minimum 3 control plane to have high availability setup, so we will start first by setting up the first node:

- Download the software for air gaped environment

```bash
cat /home/admin/Downloads/script.sh 
mkdir ./rke2-artifacts && cd ./rke2-artifacts/
curl -OLs https://github.com/rancher/rke2/releases/download/v1.33.6%2Brke2r1/rke2-images.linux-amd64.tar.zst
curl -OLs https://github.com/rancher/rke2/releases/download/v1.33.6%2Brke2r1/rke2.linux-amd64.tar.gz
curl -OLs https://github.com/rancher/rke2/releases/download/v1.33.6%2Brke2r1/sha256sum-amd64.txt
curl -sfL https://get.rke2.io --output install.sh
```

- add  configuration on first RKE2 Master Node as follows

```
cat /etc/rancher/rke2/config.yaml 
node-name: rke2-m01
server: https://127.0.0.1:9234
cluster-init: true
tls-san:
  - 10.10.21.143
  - 10.10.21.140
  - 10.10.21.141
  - 10.10.21.142
  - rke2-m01
  - rke2-m02
  - rke2-m03
  - rke2-ha
  - rke2-lb01

write-kubeconfig-mode: "0644"
```

- Enable Start & monitor the RKE2 service 

```
systemctl enable --now rke2-server 

tail -f /var/log/messages

```


###  Configure Second & Third Control Planes

```
cat /etc/rancher/rke2/config.yaml 
node-name: rke2-m02
token: 
server: https://rke2-ha:9345
tls-san:
  - 10.10.21.143
  - 10.10.21.140
  - 10.10.21.141
  - 10.10.21.142
  - rke2-m01
  - rke2-m02
  - rke2-m03
  - rke2-ha
  - rke2-lb01

write-kubeconfig-mode: "0644"
```

```
cat /etc/rancher/rke2/config.yaml 
node-name: rke2-m03
token: 
server: https://rke2-ha:9345
tls-san:
  - 10.10.21.143
  - 10.10.21.140
  - 10.10.21.141
  - 10.10.21.142
  - rke2-m01
  - rke2-m02
  - rke2-m03
  - rke2-ha
  - rke2-lb01

write-kubeconfig-mode: "0644"
```

## Configure Agents (Workers)

### Create Configuration File

```
cat /etc/rancher/rke2/config.yaml 
node-name: rke2-w01
token: 
server: https://rke2-ha:9345
write-kubeconfig-mode: "0644"
```

