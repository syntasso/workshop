This is Part 1 of [a series](./README.md) illustrating how Kratix works. 
* Up next: [Quick Start: Install a Kratix Promise](/installing-a-promise/)

<hr>


# What is Kratix?
<img alight="right" src="../assets/images/white_logo_color_background_300.jpeg" alt="Kratix logo" >

Kratix is a framework that enables co-creation of capabilities by providing a clear contract between application and platform teams through the definition and creation of “Promises”. Using the GitOps workflow and Kubernetes-native constructs, Kratix provides a flexible solution to empower your platform team to curate an API-driven, bespoke platform that can easily be kept secure and up-to-date, as well as evolving as business needs change.

## Promises:
- provide the right abstractions to make your developers as productive, efficient, and secure as possible. Any capability can be encoded and delivered via a Promise, and once “Promised” the capability is available on-demand, at scale, across the organisation.
- codify the contract between platform teams and application teams for the delivery of a specific service, e.g. a database, an identity service, a supply chain, or a complete development pipeline of patterns and tools.
- can be shared and reused between platforms, teams, business units, even other organisations.
- are easy to build, deploy, and update. Bespoke business logic can be added to each Promise’s pipeline.
- can create “Workloads”, which are deployed, via the GitOps Toolkit, across fleets of Kubernetes clusters.

## Promise anatomy
A Promise is comprised of three elements:
- Custom Resource Definition: input from an app team to create instances of a capability.
- Worker Cluster Resources: dependencies necessary for any created Workloads.
- Request Pipeline: business logic required when an instance of a capability is requested.

Now that you know more about Kratix, let's install Kratix locally.

<hr>

## Installing a multi-cluster Kratix using KinD

### Prerequisites:
1. **Kubernetes-in-Docker(KinD)**: see [the quick start guide](https://kind.sigs.k8s.io/docs/user/quick-start/). Tested on 0.9.0 and 0.10.0.
1. Ensure no KinD clusters are currently running.<br>
  `kind get clusters`<span>&nbsp;</span><span>&nbsp;</span> should return<span>&nbsp;</span><span>&nbsp;</span> `No kind clusters found.`<br>
  Any existing clusters can be deleted using the<span>&nbsp;</span><span>&nbsp;</span> `kind delete`<span>&nbsp;</span><span>&nbsp;</span> command
1. **kubectl**: see [the install guide](https://kubernetes.io/docs/tasks/tools/#kubectl). Tested on 1.16.13 and 1.21.2.
1. A **Docker Hub account** with push permissions.
1. **[Docker CLI](https://docs.docker.com/get-docker/)** to build and push images.

### Configure Docker
In order to complete all tutorials in this series, you must allocate enough resources to Docker.

Docker requires:
* 5 CPU
* 12GB Memory
* 4GB swap

This can be managed through your tool of choice (e.g. Docker Desktop, Rancher, etc).

### Clone Kratix
```
git clone https://github.com/syntasso/kratix.git
```

### Set up your platform cluster

The below commands will create our platform cluster and install Kratix.

```
kind create cluster --name platform
kubectl apply -f distribution/kratix.yaml
kubectl apply -f hack/platform/minio-install.yaml
```

The Kratix API should now be available.

```
kubectl get crds
```

The above command will give an output similar to:
```
NAME                                     CREATED AT
clusters.platform.kratix.io   2022-05-10T11:10:57Z
promises.platform.kratix.io   2022-05-10T11:10:57Z
works.platform.kratix.io      2022-05-10T11:10:57Z
```

### Multi-Cluster Networking
Some KinD installations use non-standard networking. To ensure cross-cluster communication we need to run this script: 

```
PLATFORM_CLUSTER_IP=`docker inspect platform-control-plane | grep '"IPAddress": "172' | awk '{print $2}' | awk -F '"' '{print $2}'` 
sed -i'' -e "s/172.18.0.2/$PLATFORM_CLUSTER_IP/g" hack/worker/gitops-tk-resources.yaml
```

### Set up Worker Cluster
This will create a cluster for running the X-as-a-service workloads:

```
kind create cluster --name worker #Also switches kubectl context to worker
kubectl apply -f config/samples/platform_v1alpha1_worker_cluster.yaml --context kind-platform #register the worker cluster with the platform cluster
kubectl apply -f hack/worker/gitops-tk-install.yaml
kubectl apply -f hack/worker/gitops-tk-resources.yaml
```

Once Flux is installed and running (this may take a few minutes), the Kratix resources will be visible on the worker cluster.

```
kubectl get ns kratix-worker-system
```

The above command will give an output similar to:
```
NAME                   STATUS   AGE
kratix-worker-system   Active   4m2s
```

Congratulations! Kratix is now installed.
