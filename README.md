# Reactionetes

Spin up a Kubernetes stack dedicated to Reaction Commerce PDQ

[![Build Status](https://travis-ci.org/joshuacox/reactionetes.svg?branch=master)](https://travis-ci.org/joshuacox/reactionetes) [![CircleCI](https://circleci.com/gh/joshuacox/reactionetes/tree/master.svg?style=svg)](https://circleci.com/gh/joshuacox/reactionetes/tree/master)
[![Waffle.io - Columns and their card count](https://badge.waffle.io/joshuacox/reactionetes.svg?columns=all)](https://waffle.io/joshuacox/reactionetes)

## Oneliner Autopilot

The oneliner:
```
curl -L https://git.io/reactionetes | bash
```

That merely performs the
[autopilot](#autopilot)
recipe using the [bootstrap](./bootstrap) file.

This script automagically attempts to determine your OS and tries to
install the minikube and kubectl appropriate for your OS.

At the end of which you will get some notes if everything went
successfully.  Here is some example output:

```
NOTES:
1. Get the ReactionCommerce URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l
"app=reactionetes,release=EXAMPLE_RELEASE" -o
jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:3001 to use your application"
  kubectl port-forward $POD_NAME 3001:3000
```

It should be noted that you can then paste the last three lines from
your output (not the above example lines, `EXAMPLE_RELEASE` will have your
release name that is generated by kubernetes instead)
directly into your terminal and you will be able to access the reaction
commerce site once all the pods spin up at:

[127.0.0.1:3001](http://127.0.0.1:3001)

it may take some time depending mainly on your internet connection and
how fast you can download all the necessary images.  The first time
being the worst as you have to download kubectl and minikube, AND all
the docker images to spin up kubernetes.

### ENV VARS

there are a few environment variable you can set beforehand as well

```
export MINIKUBE_OPTS=--vm-driver=none
export REACTIONETES_NAME=my-release-name
export REPLICAS=3
export MONGO_REPLICAS=5
curl -L https://git.io/reactionetes | bash
```

## Manual Installation

### Requirements

[Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to access your cluster

A running kubernetes stack, perhap using
[minikube](https://github.com/kubernetes/minikube)

with tiller initialized by
[helm](https://helm.sh/)

helm should be able to talk to your k8s cluster
and install freely

#### Windows

On
Windows
[Chocolatey](https://chocolatey.org/)
can simplify installation

#### Mac OS X

On
Macintosh
[homebrew](https://brew.sh/)
can simplify installation

### [Install](https://docs.helm.sh/helm/#helm-install)

the helm  [Install](https://docs.helm.sh/helm/#helm-install) command can
be used like this:

```
helm install ./reactionetes
```

or you can name the release

```
helm install --name my-release-name ./reactionetes
```

## Autopilot

This will
1. install minikube, kubectl, and helm
1. startup a cluster minikube,
1. config kubectl to use the cluster,
1. initialize tiller using helm,
1. spin up the mongo cluster
1. finally run a reaction pod and connect it to the mongo cluster


```
make autopilot
```

or it can be done in a one-off sort of manner using the oneliner:

```
curl -L https://git.io/reactionetes | bash
```

or specify your own minikube opts:

```
export MINIKUBE_OPTS=--vm-driver=kvm
curl -L https://git.io/reactionetes | bash
```

or on a VM that has docker installed you can run without the
virtualization driver

```
export MINIKUBE_OPTS=--vm-driver=none
curl -L https://git.io/reactionetes | bash
```

## [values.yaml](./reactionetes/values.yaml) Config

You can easily swap out your image by altering these lines:
[image settings](./reactionetes/values.yaml#L5-L7)

And the external host using these lines:
[host setting](./reactionetes/values.yaml#L17-L18)

These values can be overridden on the command line usingi the `--set` and
`--values` flags for helm, more info
[here](https://docs.helm.sh/helm/#helm-install)
and [here](https://docs.helm.sh/using_helm/#using-helm)

## Scaling

`helm list` will give you the RELEASE-NAME if you did not specify it,
once you have this you can scale your deployment:

```
kubectl scale --replicas=3 deployment/RELEASE-NAME-reactionetes
```

or you can specify a larger scale on `helm install`

```
helm install --name my-release-name --set replicaCount=30 --set mongoReplicaCount=100 ./reactionetes
```

or even as environment variables before calling make:

```
REACTIONETES_NAME=my-release-name REPLICAS=3 MONGO_REPLICAS=5 make
```

## Debug

```
helm install --dry-run --debug ./reactionetes > /tmp/manifest
```


## Makefile

#### install minikube

minikube, kubctl, and helm are updated often, it can't hurt to run this accordingly

```
make reqs
```


#### helm install .

this is the default for this makefile

```
make
```

#### Debug

you can also save a debug copy of what manifest file will be generated
using:

```
make debug
```

This will produce output pointing you to the saved manifest file in your
temp directory:

```
make debug
helm install --dry-run --debug . > /tmp/tmp.zZGCOwCCoqDOCKERTMP/manifest
ls -lh /tmp/tmp.zZGCOwCCoqDOCKERTMP/manifest
-rw-r--r-- 1 thoth thoth 5.0K Nov 22 15:44
/tmp/tmp.zZGCOwCCoqDOCKERTMP/manifest
```


#### Perftest

```
make timeme
```

How long to spin up the cluster?
Travis-CI builds it in just over two minutes:

```
Wait on Reactionetes to become available.....
hissing-manta-reactionetes-75ccf68d59-jsdrc   0/1       Running   3          1m
Reactionetes is now up and running.
	Command being timed: "bash ./bootstrap"
	User time (seconds): 11.17
	System time (seconds): 1.78
	Percent of CPU this job got: 9%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 2:18.34
	Average shared text size (kbytes): 0
	Average unshared data size (kbytes): 0
	Average stack size (kbytes): 0
	Average total size (kbytes): 0
	Maximum resident set size (kbytes): 50612
	Average resident set size (kbytes): 0
	Major (requiring I/O) page faults: 118
	Minor (reclaiming a frame) page faults: 172368
	Voluntary context switches: 19451
	Involuntary context switches: 16907
	Swaps: 0
	File system inputs: 25184
	File system outputs: 1280448
	Socket messages sent: 0
	Socket messages received: 0
	Signals delivered: 0
	Page size (bytes): 4096
	Exit status: 0
```

This now waits on all three mongo containers to spin up and the
reactionetes container to be 'Running'.

## branches

#### deprecated
These are stale and should only be looked at as an example

The master branch should work fine to test out Reaction on a local
minikube setup.  There will be other branches for other setups
and cloud providers.

#### [gce-ssd](https://github.com/joshuacox/reactionetes/tree/gce-ssd)

This branch will use a SSD GCE persistent disk on the google cloud platform

#### [gce-hdd](https://github.com/joshuacox/reactionetes/tree/gce-hdd)

This branch will use a HDD GCE persistent disk on the google cloud platform


## Notes:

Kubernetes blog [post](http://blog.kubernetes.io/2017/01/running-mongodb-on-kubernetes-with-statefulsets.html) from  January 2017

Mongodb blog [post](https://www.mongodb.com/blog/post/running-mongodb-as-a-microservice-with-docker-and-kubernetes)

KubernetesUpAndRunning examples [repo](https://github.com/kubernetes-up-and-running/examples)


### todo

add aws support

add openebs support

add a production setup that utilizes a pre-existing mongo stack and has
many different reaction deployments

add kadira

add snowplow

add benchmarking

combine all branches using go templating conditionals
