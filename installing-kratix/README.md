This is Part 1 of [a series](../README.md) illustrating how Kratix works. <br/>
👉🏾&nbsp;&nbsp; Next: [Quick Start: Install a Kratix Promise](/installing-a-promise/)

<hr>

### In this tutorial, you will
* [learn more about Kratix as a framework](#what-is-kratix)
* [install a multi-cluster Kratix using KinD](#install-kratix)

# <a name="what-is-kratix"></a> What is Kratix?

<img
  align="right"
  src="../assets/images/logo_300_with-padding.png"
  alt="Kratix logo"
/>

Kratix is a framework used by platform teams to build the custom platforms tailored to their organisation.

## By using Kratix to build your platform, you can:

* use GitOps workflow and familiar Kubernetes-native constructs.
* co-create capabilities by providing a clear contract between application and platform teams through the definition and creation of “Promises”. We'll talk more about Kratix Promises in [the next step](/installing-a-promise/README.md).
* create a flexible platform with your Golden Paths as Promises.
* evolve your platform easily as your business needs change. 
* start small on a laptop and expand to multi-team, multi-cluster, multi-region, and multi-cloud with a consistent API everywhere.

## Providing a Kratix-built platform allows your users to:
- discover available services that are already fit-for-purpose.
- consume services on demand using standard Kubernetes APIs.
- move focus away from infrastructure toward adding product value.

Now that you know more about Kratix, let's install Kratix locally.

<br>
<hr>
<br>

## <a name="install-kratix"></a> Quick Start: installing a multi-cluster Kratix using KinD

### Install Kratix prerequisites
1. **Kubernetes-in-Docker(KinD)**: see [the quick start guide](https://kind.sigs.k8s.io/docs/user/quick-start/).

1. Ensure no KinD clusters are currently running.<br>
  `kind get clusters`<span>&nbsp;</span><span>&nbsp;</span> should return<span>&nbsp;</span><span>&nbsp;</span> `No kind clusters found.`<br>
  Any existing clusters can be deleted using the<span>&nbsp;</span><span>&nbsp;</span> `kind delete`<span>&nbsp;</span><span>&nbsp;</span> command

1. **kubectl**: see [the install guide](https://kubernetes.io/docs/tasks/tools/#kubectl).

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

The below commands will create your platform cluster and install Kratix.

```
kind create cluster --name platform
kubectl apply -f distribution/kratix.yaml
kubectl apply -f hack/platform/minio-install.yaml
```

The Kratix API is now available.

```
kubectl get crds
```

The above command will give an output similar to:
```
NAME                                   CREATED AT
clusters.platform.kratix.io            2022-05-10T11:10:57Z
promises.platform.kratix.io            2022-05-10T11:10:57Z
workplacements.platform.kratix.io      2022-05-10T11:10:57Z
works.platform.kratix.io               2022-05-10T11:10:57Z
```

### Adjust multi-cluster networking for KinD
Some KinD installations use non-standard networking. To ensure cross-cluster communication you need to run this script:

```bash
PLATFORM_CLUSTER_IP=`docker inspect platform-control-plane | grep '"IPAddress": "172' | awk '{print $2}' | awk -F '"' '{print $2}'`
sed -i'' -e "s/172.18.0.2/$PLATFORM_CLUSTER_IP/g" hack/worker/gitops-tk-resources.yaml
```

### Set up your worker cluster
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

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; Kratix is now installed. <br/>
👉🏾&nbsp;&nbsp; Let's [install an off-the-shelf Kratix Promise](/installing-a-promise/README.md).
