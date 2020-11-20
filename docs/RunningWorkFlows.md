
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

Create YAML extract to patch on-line demo

```
cat <<END > repo.yaml
spec:
  artifactRepositoryRef:
    key: repo1
END

```

Now run the workflow, merging in the repo snippet

```
curl -sL https://raw.githubusercontent.com/argoproj/argo/master/examples/artifact-passing.yaml | yq m - repo.yaml | argo submit --watch -
```

## Argo Events

In this example we're going to run a Nats based eventbus. This event bus can be shared by multiple eventsources and sensors 

```
kubectl apply -f examples/argo-events/nats-eventbus.yaml
```

Setup an Eventsource and Sensor which will trigger a Workflow

```
kubectl apply -f examples/argo-events/minio-eventsource.yaml
kubectl apply -f examples/argo-events/minio-sensor.yaml
```

Now upload a file into the repo2 bucket in minio

* http://minio.test/minio/repo2/

And observe how this triggers a workflow automatically

* http://argo.test

