<<<<<<< HEAD
# argo-workflow-demo
A demo app describing how to setup and use Argo Workflows
=======
# Argo Workflows

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

TODO

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
>>>>>>> Init
