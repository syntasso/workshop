# Kratix Multi-Cluster Install
_To install on a single cluster use the [Single Cluster Quick Start Guide](https://github.com/syntasso/kratix/blob/main/docs/single-cluster.md)._

### Prerequisites:
1. **Kubernetes-in-Docker(KinD)** See [the quick start guide](https://kind.sigs.k8s.io/docs/user/quick-start/). Tested on 0.9.0 and 0.10.0.
    - Ensure no KinD clusters are currently running. `kind get clusters` should return "No kind clusters found."
1. **Kubectl** See [the install guide](https://kubernetes.io/docs/tasks/tools/#kubectl). Tested on 1.16.13 and 1.21.2.
1. A **Docker Hub account** with push permissions (or similar registry).
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

### Set up Platform Cluster

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

## Feedback

Please help to improve Kratix. Give feedback [via email](mailto:feedback@syntasso.io?subject=Kratix%20Feedback) or [google form](https://forms.gle/WVXwVRJsqVFkHfJ79). Alternatively, open an issue or pull request.
