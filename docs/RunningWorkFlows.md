
Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Running workflows](#running-workflows)
      * [Hello world](#hello-world)
      * [Artifact passing](#artifact-passing)
      * [Argo Events](#argo-events)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

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
data:
  minio: |
    s3:
      bucket: demo2
      endpoint: minio.minio:9000
      insecure: true
      accessKeySecret:
        name: my-minio-cred
        key: accessKey
      secretKeySecret:
        name: my-minio-cred
        key: secretKey
---
apiVersion: v1
kind: Secret
metadata:
  name: my-minio-cred
stringData:
  accessKey: XXXXXXXXXX
  secretKey: YYYYYYYYYY
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
curl -sL https://raw.githubusercontent.com/argoproj/argo/master/examples/artifact-passing.yaml | yq m - repo.yaml | argo submit --watch -
```

## Argo Events

Create a default eventbus locally

```
kubectl apply -f examples/argo-events/nats-eventbus.yaml
```

Create an EventSource and secret

```
kubectl apply -f examples/argo-events/my-minio-cred-secret.yaml
kubectl apply -f examples/argo-events/minio-eventsource.yaml
```

Create a Sensor which runs a workflow 

```
kubectl apply -f examples/argo-events/minio-sensor.yaml
```

