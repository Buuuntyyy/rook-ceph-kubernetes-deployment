# rook-ceph-kubernetes-deployment
A tutorial to deploy a functionnal and ready to use [Rook-ceph](https://github.com/rook/rook) cluster in your kubernetes infrastructure.
Many already exists, but after many days of experimentation, I decide to read my own, such as others are not fully explicit (especially the doc about kubernetes).

Rook-ceph is a combination of 2 technologies mainly used for cloud and storage. Here are their concept :
1. Rook
Rook is an operator for Kubernetes which aims to make easy to implement storage solution inside a cluster.
2. Ceph
Ceph is an openSource Storage System that gives Object, Block and Files storage solution. It is highly scalable and reliable.

 1. Deploy rook
There are 2 ways to deploy rook :
* Helm charts (RookCeph operator and RookCeph cluster)
* The Quickstart Guide (through a git clone and examples files).

# Requirements
Kubernetes versions v1.28 through v1.33 are supported.
Architectures released are amd64 / x86_64 and arm64.
A Kubernetes Cluster with 4 nodes (1 master and 3 workers)
3 no formatted filesystem disks (no partition) beside your system disk
⚠️For this tutorial, kubelet v1.33 is used.

# QuickStart Guide
The quickstart guide is really easy to follow, but it will only enable you to have a cluster running (HEALTH_OK), without any pools, metadataServers...
The main issue is that, even though there is a huge [documentation](https://rook.io/docs/rook/latest-release/Getting-Started/intro/), it doesn't show how to use every files enable in the git repo. You have to search and understand by yourself for everything you want to add, and all dependancies.

## Install Rook-ceph following quickstart
You can use the original files, but you can use mine to and then, custom according to your infrastructure.
``` bash
git clone --single-branch --branch v1.17.7 https://github.com/rook/rook.git
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
kubectl create -f cluster.yaml
```
### Details
* common.yaml contains the shared resources by all rook components (eg. Namespaces ServiceAccounts ClusterRoles ClusterRoleBindings CustomResourceDefinitions PodSecurityPolicies)
* crds.yaml is the file with custom resources definition (eg. CephCluster CephBlockPool CephFilesystem CephObjectStore CephClientCephFilesystemSubVolumeGroupCephNFS)
* operator.yaml launch the Operator that is the main controller of the cluster

## Create the Ceph cluster
Before crating the cluster.yaml resource, wait for the operator pod to be in Running state (1/1)
``` bash
kubectl get pods -n rook-ceph
```
Then, create the CephCluster
``` bash
kubectl create -f cluster.yaml
```
### Details
* cluster.yaml contains a CephCluster Custom Resource (CR) that describe the entire cluster configuration to deploy via Rook

## Cluster status
Now, we have to wait for the CephCluster to be entirely deployed. You can check its current status by :
``` bash
kubectl get cephcluster -n rook-ceph
```
Once it reaches the "Cluster Created Successfuly" status, check your pods :
``` bash
kubectl get pods -n rook-ceph
```
Output should looks like :
``` bash
NAME                                                     READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-ghss4                                   3/3     Running     0          24h
csi-cephfsplugin-provisioner-cb4fc4fff-zbx8l             6/6     Running     0          24h
csi-cephfsplugin-provisioner-cb4fc4fff-zt9t5             6/6     Running     0          24h
csi-cephfsplugin-tmx49                                   3/3     Running     0          24h
csi-cephfsplugin-xbtjc                                   3/3     Running     0          24h
csi-rbdplugin-cm8hr                                      3/3     Running     0          24h
csi-rbdplugin-plxkp                                      3/3     Running     0          24h
csi-rbdplugin-provisioner-bdfc6844c-2sfd5                6/6     Running     0          24h
csi-rbdplugin-provisioner-bdfc6844c-q7jwb                6/6     Running     0          24h
csi-rbdplugin-sfllt                                      3/3     Running     0          24h
rook-ceph-crashcollector-dev-worker-1-79fbf5744f-9kbjw   1/1     Running     0          24h
rook-ceph-crashcollector-dev-worker-2-5bf896df9d-7lfsx   1/1     Running     0          24h
rook-ceph-crashcollector-dev-worker-3-88b5ddc67-qjzd5    1/1     Running     0          24h
rook-ceph-exporter-dev-worker-1-5c687654f4-5mgtj         1/1     Running     0          24h
rook-ceph-exporter-dev-worker-2-84f4d69c57-vd5z2         1/1     Running     0          24h
rook-ceph-exporter-dev-worker-3-5f4d76d796-mp8cz         1/1     Running     0          24h
rook-ceph-mgr-a-5884988474-mhzzb                         3/3     Running     0          24h
rook-ceph-mgr-b-54fdb97569-lt77p                         3/3     Running     0          24h
rook-ceph-mon-a-f8bc48cb-wzfjk                           2/2     Running     0          24h
rook-ceph-mon-b-59c9c7f5c5-68krc                         2/2     Running     0          24h
rook-ceph-mon-c-5469f9d8f7-s4dq4                         2/2     Running     0          24h
rook-ceph-operator-5bd54f7b65-pcflb                      1/1     Running     0          25h
rook-ceph-osd-0-f977ff9b8-5xb25                          2/2     Running     0          24h
rook-ceph-osd-1-5fbfc46b5-c5cqb                          2/2     Running     0          24h
rook-ceph-osd-2-55ddbb5cc8-cxr67                         2/2     Running     0          24h
rook-ceph-osd-prepare-dev-worker-1-kzmk5                 0/1     Completed   0          24h
rook-ceph-osd-prepare-dev-worker-2-5nnm6                 0/1     Completed   0          24h
rook-ceph-osd-prepare-dev-worker-3-wzdln                 0/1     Completed   0          24h
```
## Ceph in Kubernetes
There are several way to execute Ceph commands for kubernetes. You can deploy a [Toolbox](https://rook.io/docs/rook/latest-release/Troubleshooting/ceph-toolbox/) or install a [kubectl plugin](https://rook.io/docs/rook/latest-release/Troubleshooting/kubectl-plugin/) to manage everything from your current CLI. For this tutorial we will use the second purpose.

### Install Ceph kube plugin
In order to install ceph kubectl plugin, we need to first install krew on our system, from its git repo
``` bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```
Then, you can export the variable to your path by executing the following command, or adding it yo your bashrc
``` bash
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```
ℹ️ After adding a command to your bashrc, don't forget to restart your shell

Now, you can install the ceph kubectl plugin, and execute the status command from your CLI
``` bash
kubectl krew install rook-ceph
kubectl rook-ceph ceph status
```
The output should looks like
``` bash
  cluster:
    id:     ...
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c (age 7m)
    mgr: a(active, since 6m), standbys: b
    osd: 3 osds: 3 up (since 5m), 3 in (since 6m)

  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 577 KiB
    usage:   81 MiB used, 450 GiB / 450 GiB avail
    pgs:     1 active+clean
```
⚠️⚠️⚠️ If, at this step, your cluster health is not HEALTH_OK, then stop this tutorial and go to the official [troubleshooting documentation](https://rook.io/docs/rook/latest-release/Troubleshooting/common-issues/), follow the official [cleanup tutorial](https://rook.io/docs/rook/latest-release/Getting-Started/ceph-teardown/) or mine at the end of this installation tutorial. ⚠️⚠️⚠️
⚠️ **From the ceph status, you can see that there is only 1 pool and 1 pg configured. This pool and its attached pg are only for management and doesn't aims to store data.** ⚠️

# Storage configuration
Now that the cluster is healthy, you can configure your storage. Remember that are required :
* 3 unformatted filesystem disks
* 3 worker nodes

Each worker-node is represented by an OSD pod, if you have less OSD pods than node (excepted for custom installation, there is an error and Rook probably don't see the disk linked to your node). Once again, check the [official troubleshooting documentation](https://rook.io/docs/rook/latest-release/Troubleshooting/common-issues/)

## Storage type
# 🧊 Rook Ceph Storage Types in Kubernetes

Rook Ceph provides **three main types of storage**, each suitable for different use cases:

---

## 🔹 1. `rook-ceph-block` — **Block Storage**

- **Type:** Block (RBD - RADOS Block Device)  
- **Access Mode:** `ReadWriteOnce` (RWO)  
- **StorageClass:** `rook-ceph-block`

### ✅ Use Cases:
- Databases (PostgreSQL, MySQL, etc.)
- Applications needing **low-latency persistent storage**
- One pod per volume

### ⚙️ How It Works:
- Creates a **virtual disk** (block device)
- Formatted and mounted inside the pod (ext4, xfs, etc.)

### ⚠️ Limitations:
- Not shareable between pods

---

## 🔹 2. `rook-cephfs` — **Shared File System**

- **Type:** POSIX-compliant shared file system  
- **Access Mode:** `ReadWriteMany` (RWX)  
- **StorageClass:** `rook-cephfs`

### ✅ Use Cases:
- Web applications needing **shared file access**
- Nextcloud `data/`, static files
- Configs or media used by multiple pods

### ⚙️ How It Works:
- Uses **CephFS** (via Ceph Metadata Server - MDS)
- Classical POSIX file system, accessible by multiple clients

### ⚠️ Limitations:
- Slightly higher latency
- MDS components need tuning and monitoring

---

## 🔹 3. **CephObjectStore** — **S3-Compatible Object Storage**

- **Type:** Object storage (buckets)  
- **Access Mode:** Not mountable (accessed via HTTP[S] API)  
- **Access Via:** S3-compatible clients (Rclone, MinIO Gateway…)

### ✅ Use Cases:
- Static files (images, backups…)
- Logs, archives
- Direct access via API

### ⚙️ How It Works:
- Creates an **S3-compatible endpoint** (via Ceph RGW)
- Access via AWS CLI, Rclone, etc.

### ⚠️ Limitations:
- Cannot be used as a PVC
- Client application must handle S3 API logic

---

## 🔄 Summary Table

| Feature                    | `rook-ceph-block`        | `rook-cephfs`               | CephObjectStore (S3)       |
|---------------------------|--------------------------|-----------------------------|-----------------------------|
| **Type**                  | Block (RBD)              | Shared file system          | Object (S3 API)             |
| **Mountable in Pod**      | ✅                       | ✅                          | ❌                          |
| **Shared Access**         | ❌                       | ✅                          | ✅ (via API)                |
| **POSIX Support**         | ✅ (via formatting)      | ✅                          | ❌                          |
| **Kubernetes PVC Support**| ✅                       | ✅                          | ❌                          |
| **Typical Use**           | Database volumes          | Nextcloud shared files      | Backups, media              |
| **CSI Provisioner**       | RBD CSI                  | CephFS CSI                  | N/A                         |

---

## ✅ Recommendations

- 🛢️ Use **`rook-ceph-block`** for low-latency apps and databases  
- 📂 Use **`rook-cephfs`** for multi-pod shared storage (Nextcloud, web apps)  
- 🌐 Use **CephObjectStore** for S3-style object access (backups, large files)

## CephBlockPool configuration
As shown above, if you have a kubernetes instance, you will mainly be interested by deploying CephBlockPool in order to provision your pods RWO PVCs'.
### Create a CephBlockPool configuration file
You will find this configuration file in the [official storage documentation](https://rook.io/docs/rook/latest-release/Storage-Configuration/Block-Storage-RBD/block-storage/#provision-storage).
You can also take the one in this repo.
After creating the file, apply it to your cluster :
``` bash
kubectl apply -f CephBlockPool.yaml
```
### Create a CephBlock storageclass for your PVCs
You will find this configuration file in the [official storage documentation](https://rook.io/docs/rook/latest-release/Storage-Configuration/Block-Storage-RBD/block-storage/#provision-storage).
You can also take the one in this repo.
After creating the file, apply it to your cluster :
``` bash
kubectl apply -f storageclass-cephblock.yaml
```

Check that both files have been well applied to your cluster
* For the Pool, you can check the cephcluster status, and list your pools. Your new block pool should appear.
* ``` bash
  kubectl rook-ceph ceph status
  kubectl rook-ceph ceph pool ls
  ```
``` bash
kubectl get storageclass
```
### Create a CephFS configuration file
You will find this configuration file in the [official storage documentation](https://rook.io/docs/rook/latest-release/Storage-Configuration/Shared-Filesystem-CephFS/filesystem-storage/).
You can also take the one in this repo.
After creating the file, apply it to your cluster :
``` bash
kubectl apply -f CephFSPool.yaml
```
### Create a CephFS storageclass for your PVCs
You will find this configuration file in the [official storage documentation](https://rook.io/docs/rook/latest-release/Storage-Configuration/Shared-Filesystem-CephFS/filesystem-storage/).
You can also take the one in this repo.
After creating the file, apply it to your cluster :
``` bash
kubectl apply -f storageclass-cephFS.yaml
```

Check that both files have been well applied to your cluster
* For the Pool, you can check the cephcluster status, and list your pools. Your new block pool should appear. Thus, new MDS pods should have been created.
* ``` bash
  kubectl get pods -n rook-ceph
  kubectl rook-ceph ceph status
  kubectl rook-ceph ceph pool ls
  ```
``` bash
kubectl get storageclass
```
**You now have Rook-ceph fully deployed and ready-to-use ! Don't forget to set the correct storageclass for your future deployments**
# Dashboard configuration
A dashboard is enabled by default
``` bash
kubectl get svc -n rook-ceph
```
To access it from your Node IP, you can deploy the related LoadBalancer (eg. if you have a dedicated reverse proxy), or an ingress
``` bash
kubectl apply -f deploy/examples/dashboard-loadbalancer.yaml
```

# Tricks
## Set the Block storageclass as default
In case your forget to set the storageclass in your deployment, you can configure the Block storageclass (RWO) as default
``` bash
kubectl patch storageclass ceph-block \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
