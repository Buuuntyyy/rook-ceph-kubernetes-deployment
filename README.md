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

## Wait for 
