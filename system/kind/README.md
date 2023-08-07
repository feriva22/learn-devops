# How to use KIND (Kubernetes In Docker) to deploy kubernetes inside docker

## Getting Started
We will use kind as alternative for Minikube

### 1. Install kind command line
```
apt install golang-go
go install sigs.k8s.io/kind@v0.20.0
```
Add go env to $PATH
```
export PATH=$PATH:$(go env GOPATH)
```
### 2. Create first cluster
```
kind create cluster
```
This is will create Single-node cluster


## Creating cluster with file definitions yaml
We will create multiple clusters, with different name and different node count
```
kind create cluster --config cluster/development-cluster.yaml
kind create cluster --config cluster/staging-cluster.yaml
kind create cluster --config cluster/production-cluster.yaml
```