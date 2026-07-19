
# KUBERNETES

## POD:

- smallest unit in Kubernetes 
- Abstraction over containers.
- Usually 1 Application.
- Each Pod get it's own IP Address.
- Pod fails, new IP generated and assigned to that pod after restart it.
> to solve the IP regeneration we can use service component in kubernetes that depends on the name instead of IP.

## Service & Ingress:

- Permanent IP address.
- Life cycle of pod & services not connected. 
- two type of services:
	- External Services: like web page through the browser.
	- internal services: like db accept connection form app PODs.
## ConfigMap & Secret:

- External config of my-application.
- Store creds inside the secret like db creds. 
## Volume:

- store data locally on same POD Node.
- Store data remotely outside the POD node.

## Deployment & StatefulSet:

- service has 2 functions:
	- permanent IP.
	- Load balance 
- blueprint for my app-pod.
- create deployment instead of pod.
- Abstraction of pod.
- StatefulSet (sts) same as deployment but for database services, which priority is consistency.

## Kubernetes Architecture:

### Node Processes:

**Kubelet:-** 

The **kubelet** is an agent that runs on each node in the Kubernetes cluster. Its main responsibilities include:

- **Managing Pods**: It ensures that containers described in PodSpecs are running and healthy.
- **Interacting with the container runtime** (like containerd in RKE2) to start, stop, and monitor containers.
- **Reporting node status** to the Kubernetes control plane.
- **Handling volume mounts, secrets, and config maps** for pods.

**Kube Proxy:-**

**kube-proxy** is a network component that runs on each node in a Kubernetes cluster. Its main job is to manage network communication to and from **services**. Its main Responsibilities include:

- **Service Routing**: It ensures that traffic sent to a Kubernetes service is correctly forwarded to one of the service’s backend pods.
- **Load Balancing**: Distributes traffic across multiple pod endpoints.
- **IPTables/IPVS Rules**: Sets up low-level networking rules using tools like `iptables` or `IPVS` to efficiently route traffic.
- **Supports Cluster Networking**: Works with the cluster’s CNI (Container Network Interface) plugin to maintain connectivity.
- In RKE2 `kube-proxy` is deployed as a **static pod** and is **automatically managed**.

**Container Runtime:-**

The **Container Runtime** in **RKE2** is the software responsible for running containers on each node. RKE2 uses **containerd** as its default container runtime, replacing Docker for better performance, security, and Kubernetes-native integration. Main Responsibilities are:

- **Manages container lifecycle**: Starts, stops, and monitors containers as instructed by the kubelet.
- **Handles image pulling and storage**: Downloads container images and manages them locally.
- **Provides runtime interface**: Acts as the backend for Kubernetes to run workloads securely and efficiently.


### Master Processes:

So, how do you **interact**  with this cluster? How to:
- schedule pod?
- monitor?
- re-schedule/re-start pod?
- join a new Node?

**Core Kubernetes Control Plane Components**

These are standard Kubernetes services:

- **kube-apiserver** → The central API server that all clients (kubectl, controllers, nodes) talk to.
- **etcd** → The key-value store holding the cluster state (can be external or embedded).
- **kube-scheduler** → Schedules pods to nodes based on resource requests, constraints, and policies.
- **kube-controller-manager** → Runs controllers that handle replication, endpoints, namespace lifecycle, etc.    
- **cloud-controller-manager** (if cloud integration is enabled) → Manages cloud-specific resources (load balancers, volumes, routes).

**RKE2-Specific & Infrastructure Components**

- **rke2-server** → The RKE2 server agent that runs the control plane components as static pods.    
- **containerd** → The container runtime used to run workloads and system pods (RKE2 defaults to containerd, not Docker).
- **CoreDNS** → Provides internal DNS resolution for services and pods.
- **kube-proxy** → Handles networking rules on the nodes for Services.
- **cni plugin** (default: **Canal** = Flannel + Calico; but can be replaced with others like Cilium) → Manages pod networking.
**Optional / Add-On Components**

(depending on your configuration)

- **metrics-server** → Resource metrics for autoscaling (if enabled).    
- **Ingress Controller** (e.g., NGINX or Traefik, depending on setup) → Handles incoming traffic to services.
- **Service Load Balancer (klipper-lb)** → Lightweight load balancer for small/edge clusters.
- **rke2-coredns, rke2-ingress, rke2-metrics-server** → Packaged RKE2 addons deployed as Helm charts.

## KubeCTL Main Commands

```sh
# get nodes
kubectl get nodes

# get pods
kubectl get pods

# get services 
kubectl get services

# Create pod nginx in default namespace
kubectl create deployment name --image=image [--dry-rub] [opt]
kubectl create deployment nignx-dev --image=nginx

# get replicaset
kubectl get replicaset

# edit deployment on fly 
kubectl edit deployment nginx-dev

# debuging pods -f to keep follow the logs 
kubectl logs -f [pod_name]

# describe pod 
kubectl describe pod [pod_name]

# access the pod and perform actions 
kubectl exec -it [pod_name] -- bin/bash

# delete pods 
kubectl delete deployment mongo-depl
# check the deletion is success
kubectl get deployment
kubectl get replicaset 

# apply deployment from a file  
kubectl apply -f [file_name]

# delete deployment using file 
kubectl delete -f nginx-deploy.yaml

# check permissions I have on certmanager NS
kubectl auth can-i --list  --namespace=cert-manager

# check if I have create PODs on all namespaces 
kubectl auth can-i create pod  -A

```


## YAML Configuration File

YAML is human friendly data serialization standard for all programming languages and its syntax are strict indentation. 

RKE2 YAML Configuration file has three parts:
- Meta Data: details of the kind
- Specification: specs of this kind 
- Status : actual vs desired  these details taken from ETCD component 

**Label and Selector**

We can connect Services and Deployments using Label and Selector as shown below: 

```yaml
# Deployment Kind you can label a deployment 
kind: Deployment
metadata:
	name: nginx-deplo
	labels:
		app: nginx #===> this can be used in selector.
		
# you can select this label in service 
kind: Service
metadata:
	name: nginx-service
spec: 
	selector:
		app: nginx #===> this is the deplyment label.
```

## Namespaces

Here’s a clean and structured version of your notes, rewritten for clarity and optimized for use in **Obsidian** as notebook material. You can copy and paste this directly into a note:

---

**What is a Namespace?**

Namespaces in Kubernetes are used to **organize and isolate resources** within a cluster. They provide a way to divide cluster resources between multiple users or teams.

**Default Namespaces**

Kubernetes comes with **four default namespaces**:

1. **`default`**
    
    - All resources you create without specifying a namespace are placed here.
2. **`kube-node-lease`**
    
    - Used for node heartbeats.
    - Each node has a lease object in this namespace.
    - Helps determine node availability.
3. **`kube-public`**
    
    - Contains publicly accessible data.
    - Includes a `ConfigMap` with cluster information.
    - Readable by all users (even unauthenticated).
4. **`kube-system`**
    
    - Reserved for system components.
    - Includes master and `kubectl` processes.
    - ⚠️ Do **not** create or modify resources here.

**Custom Namespaces**

You can create your own namespaces to organize resources:

```Shell
kubectl create namespace my-namespace
```


**Use Cases for Namespaces**

Namespaces help manage and isolate resources effectively. Here are common use cases:

**Single Namespace for All Resources**

- Possible, but **not recommended**.
- Can lead to **conflicts** and **management difficulties**.

 **Role-Based Resource Grouping**

- Create separate namespaces for different services or roles:
    - e.g., `database`, `monitoring`, `logging`.
- Helps with organization and access control.
- Useful for **blue-green deployments**.

**Environment Separation**

- Separate namespaces for **staging**, **development**, and **production**.
- Allows shared resources to be isolated and reused across environments.

**How Namespaces Work & How to Use Them**

- Resources are scoped to namespaces.
- You can **list**, **create**, and **delete** resources within a namespace.
- Use `kubectl` commands with the `--namespace` flag to interact with specific namespaces:

```Shell
kubectl get pods --namespace=my-namespace
```

- Namespaces do **not** provide security boundaries by default — use **RBAC** for access control.
- Limit CPU RAM Storage per NS.

**Characteristics**

- you can't access most resources from anther Namespace.
	- for example if you have configmap in one NS you can use it in another NS.
	- Each Namespace should define a configmap within the namespace.
- you can share and access Service in another Namespace 
	- in configmap add the service mysql-service.database (database here is the namespace name )
	- 
- Components, which can't be created within a Namespace
	- volumes
	- nodes
- List all resources they are belong and not belong to namespace using below commands:
	- `kubectl api-resources --namespaced=false`
	- `kubectl api-resources --namespaced=true`



to change the namespace from default to any other namespace you can change the context as shown below:

```bash
kubectl config set-context --current --namespace=mongodb

# return it back 
kubectl config set-context --current --namespace=default
```


## Ingress

The Ingress can be use instead of external service, so we can expose the internal service for the web app or any public service outside the RKE2 cluster using ingress.

**External Service vs Ingress**

|  Aspect             |  External Service                                       |  Ingress                                            |
| ------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| **Purpose**         |  Expose a service externally via IP or LoadBalancer     |  Route HTTP/HTTPS traffic to multiple services      |
| **How It Works**    |  Uses `Service` type `LoadBalancer` or `NodePort`       |  Requires an Ingress Controller (e.g., NGINX)       |
| **Protocol**        |  Any protocol (TCP, UDP)                                |  Primarily HTTP/HTTPS                               |
| **IP & DNS**        |  Each service may get its own external IP               |  One external IP for all routes                     |
| **TLS/SSL**         |  Handled by app or external LB                          |  Built-in TLS termination via Ingress Controller    |
| **Complexity**      |  Simple for single service                              |  More complex but efficient for multiple apps       |
| **Use Cases**       |  Expose DB or single app externally                     |  Host multiple web apps under one domain            |

to check ingress on your kubernetes 

```shell
kubectl get ingressclass
kubectl api-resources | grep -i ingress
```

## Helm

- Main concept of Helm.
- Helm changes a lot between version, so:
	- understand the basic common principles and use cases.
Overview:

- what is helm?
	- package manager for kubernetes: 
		- to package YAML Files and distribute them in public and private repositories.
		- Bundle of YAML Files.
		- Create your own Helm Charts with helm
		- Push them to Helm Repository 
		- Download and use existing ones.
	- Templating Engine. 
		- can create a template for a specific service and reuse it on different projects.
		- important in CI/CD DevOps since I have multiple environments such as Dev, Staging, Prod.
	- Release Management:
		- Helm CLient : this send request for installation to Tiller  
		- Helm (Tiller): this will accept the request and apply the installation.
		- `helm install <> `
		- `helm upgrade <>`
		- we can use `helm rollbacks <>` to a specific version
		- 
		
- what are helm charts?
	- the Helm chart Structure:
		- the Chart Folder a for example MyChart and it will be a chart name.
		- chart.yaml, contains the meta data about the chart.
		- values.yaml values for the template files.
		- charts folder contains chart dependencies.
		- template folder contains the actual template files.
		- we can have readme.md and license files as well.
- how to use them?
	- `helm install <chartname>` values will inject auto
	- `helm install --values=my-values.yaml <chartname>` we can override value or values in that value file called `my-values.yaml` files 
- when to use them?
- what is tiller?
	- in helm v3 tiller removed because it has much power in kubernetes and it may cause security issue.

## Volumes

how to persist data in Kubernetes using volumes?

- persistent volume.
- Persistent Volume Claim.
- Storage Class.

Storage Requirements:

- storage that desn't depend on the pod lifecycle.
- storage must be available on all nodes.
- storage need to survive even if cluster crashes.

