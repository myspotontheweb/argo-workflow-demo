Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Argo Workflows](#argo-workflows)
   * [Setup](#setup)
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

## Setup Artifact repo

```
kubectl create ns minio
helm repo add minio https://helm.min.io/
helm install --namespace minio --generate-name minio/minio
```

UI

```
kubectl -n minio port-forward deployment/minio-1605732143 8002:9000
```

available at

* http://localhost:8002

Config

```
$ kubectl -n minio get secret minio-1605732143 -ogo-template='{{range $k,$v:=.data}}{{printf "%s = %s\n" $k ($v|base64decode)}}{{end}}'
accesskey = XXXXXXXXXXXXXXX
secretkey = YYYYYYYYYYYYYYY
```

# Running workflows

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
      endpoint: minio-1605732143.minio:9000
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
  accesskey: XXXXXXXXXXXXXXX
  secretkey: YYYYYYYYYYYYYYY
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

