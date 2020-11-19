Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Argo Workflows](#argo-workflows)
   * [Setup](#setup)
      * [Software](#software)
      * [Cluster](#cluster)
      * [Installation](#installation)
      * [Test](#test)
      * [CLI](#cli)
      * [Demo namespace](#demo-namespace)
      * [Setup Artifact repo](#setup-artifact-repo)
   * [Running workflows](#running-workflows)
      * [Hello world](#hello-world)
      * [Artifact passing](#artifact-passing)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

# Argo Workflows

Documentation

* [Argo workflows](https://argoproj.github.io/argo/)
* [Argo events](https://argoproj.github.io/argo-events/)
* [Argo Install manifests](https://github.com/argoproj/argo/tree/stable/manifests)

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

## Demo namespace

Working in the demo namespace

```
kubectl create ns demo
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=demo:default -n demo 
```

## Setup Artifact repo

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


# Running workflows

The following examples are all run in the demo namespace

```
kubens demo
```

## Hello world

Source workflow

* [examples/hello-world.yaml](https://github.com/argoproj/argo/blob/master/examples/hello-world.yaml)

```
argo submit --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
```

Log output

```
argo logs @latest 
```

## Artifact passing

Source workflow

* [examples/artifact-passing.yaml](https://github.com/argoproj/argo/blob/master/examples/artifact-passing.yaml)

Using the Minio UI create a bucket called demo2

The following command will create a ConfigMap and Secret pointing at the bucket in minio

```
kubectl apply -f - <<END
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: artifact-repositories
  namespace: demo
data:
  minio: |
    s3:
      bucket: demo2
      endpoint: minio.minio:9000
      insecure: true
      accessKeySecret:
        name: my-minio-cred
        key: accesskey
      secretKeySecret:
        name: my-minio-cred
        key: secretkey
---
apiVersion: v1
kind: Secret
metadata:
  name: my-minio-cred
  namespace: demo
stringData:
  accesskey: XXXXXXXXXX
  secretkey: YYYYYYYYYY
END
```

Create YAML extract that points the workflow at the minio bucket

```
cat <<END > repo.yaml
spec:
  artifactRepositoryRef:
    key: minio
END

```

Now run the workflow, merging in the repo snippet

```
curl -L https://raw.githubusercontent.com/argoproj/argo/master/examples/artifact-passing.yaml | yq m - repo.yaml | argo submit --watch -
```

