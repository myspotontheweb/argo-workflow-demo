Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Setup](#setup)
      * [Software](#software)
      * [Cluster](#cluster)
      * [Installation](#installation)
      * [Test](#test)
      * [CLI](#cli)
      * [Install Minio](#install-minio)
      * [Demo namespace](#demo-namespace)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

# Setup

## Software

The following tools need to be installed in order to run this demo

1. Docker
1. [minikube](https://minikube.sigs.k8s.io/docs/start/)
1. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
1. [helm](https://helm.sh/)
1. [kubens](https://kubectx.dev/)
1. [yq](https://mikefarah.gitbook.io/yq/)

## Cluster

```
minikube start --driver=docker --kubernetes-version=v1.16.15
```

## Installation

Argo Workflows

```
kubectl create namespace argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo/stable/manifests/install.yaml
```

Argo Events

```
kubectl create namespace argo-events
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
```

Argo eventbus examples

```
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
```

## Test

```
kubectl -n argo port-forward deployment/argo-server 8001:2746
```

WebUI available at:

* http://127.0.0.1:8001/

## CLI

```
VERSION=2.11.7

# Download the binary
curl -sLO https://github.com/argoproj/argo/releases/download/v$VERSION/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
sudo mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

## Install Minio

```
helm repo add minio https://helm.min.io/

kubectl create ns minio

cat <<END > values.yaml
accessKey: XXXXXXXXXX
secretKey: YYYYYYYYYY
buckets:
  - name: demo1
    policy: none
    purge: false
  - name: demo2
    policy: none
    purge: false
END

helm install minio minio/minio -n minio -f values.yaml
```

UI

```
kubectl -n minio port-forward deployment/minio 8002:9000
```

available at

* http://localhost:8002


## Demo namespace

Workflows need to run with additional privileges:

```
kubectl create ns demo
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=demo:default -n demo 
```

Alternatively create a dedicated service account for argo in the namespace

