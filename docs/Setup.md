
Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Setup](#setup)
      * [Software](#software)
      * [Cluster](#cluster)
      * [/etc/hosts](#etchosts)
      * [Install Minio](#install-minio)
      * [Argo Workflows   Events](#argo-workflows--events)
      * [Configure the artifact repositories](#configure-the-artifact-repositories)
      * [CLI](#cli)
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
minikube addons enable ingress
```

## /etc/hosts

Setup IP addresses for ingress end-points

```
echo "$(minikube ip) argo.test minio.test" | sudo tee -a /etc/hosts
```

## Install Minio

Add helm repo

```
helm repo add minio https://helm.min.io/
```

Helm chart custom settings

```
cat <<END > values.yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
  hosts:
  - minio.test
accessKey: XXXXXXXXXX
secretKey: YYYYYYYYYY
buckets:
- name: repo1
  policy: none
  purge: false
- name: repo2
  policy: none
  purge: false
END
```

Install minio

```
kubectl create ns minio
helm install minio minio/minio -n minio -f values.yaml
```

WebUI available at:

* http://minio.test


## Argo Workflows + Events

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

Configure UI

```
kubectl apply -f - <<END
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: argo-server-ingress
  namespace: argo
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: argo.test
    http:
      paths:
      - backend:
          serviceName: argo-server
          servicePort: web
        path: /
END
```

WebUI available at:

* http://argo.test

##  Configure the artifact repositories

```
kubectl -n argo apply -f - <<END
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: artifact-repositories
data:
  repo1: |
    s3:
      bucket: repo1
      endpoint: minio.minio:9000
      insecure: true
      accessKeySecret:
        name: artifact-repositories
        key: accessKey
      secretKeySecret:
        name: artifact-repositories
        key: secretKey
  repo2: |
    s3:
      bucket: repo2
      endpoint: minio.minio:9000
      insecure: true
      accessKeySecret:
        name: artifact-repositories
        key: accessKey
      secretKeySecret:
        name: artifact-repositories
        key: secretKey
---
apiVersion: v1
kind: Secret
metadata:
  name: artifact-repositories
stringData:
  accessKey: XXXXXXXXXX
  secretKey: YYYYYYYYYY
END
```

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

Workflows need to run with additional privileges:

```
kubectl create ns demo
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=demo:default -n demo 
```

Alternatively create a dedicated service account for argo in the namespace

