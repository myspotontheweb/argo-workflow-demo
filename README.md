Table of Contents
=================

   * [Argo Workflows](#argo-workflows)
   * [Setup](#setup)
      * [Cluster](#cluster)
      * [Installation](#installation)
      * [Test](#test)
      * [CLI](#cli)
      * [Demo namespace](#demo-namespace)
   * [Running workflows](#running-workflows)
      * [Hello world](#hello-world)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

# Argo Workflows

Documentation

* [Argo workflows](https://argoproj.github.io/argo/)
* [Argo events](https://argoproj.github.io/argo-events/)
* [Argo Install manifests](https://github.com/argoproj/argo/tree/stable/manifests)

# Setup

## Cluster

```
minikube start --driver=docker --kubernetes-version=v1.16.15
minikube addons enable ingress
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

## Demo namespace

Working in the demo namespace

```
kubectl create ns demo
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=demo:default -n demo 
```

# Running workflows

## Hello world

```
argo submit --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
```

Log output

```
argo logs @latest 
```
