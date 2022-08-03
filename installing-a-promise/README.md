This is Part 2 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Quick Start: Install Kratix](/installing-kratix/) <br/>
👉🏾&nbsp;&nbsp; Next: [Using multiple Kratix Promises](/using-multiple-promises/)

<hr>

### In this tutorial, you will
1. [learn more about Kratix Promises](#promise)
1. [install Jenkins as a Kratix Promise](#install-jenkins)

# <a name="promise"></a> What is a Kratix Promise?

## What's the Problem?
Being a Platform team member is challenging. These teams relentlessly feel tensions from many directions, and often face:
* Demands from their customers, who increasingly expect software served from internal platforms be as simple, quick to consume, and performant as commodity public-cloud services.
* Steep learning curves as they take software from large vendors, figure out how to tweak the seemingly endless configuration options to introduce the "right" level of opinions: not too opinionated to reduce utility for their users, but opinionated enough to meet their own internal SLI/SLO requirements.
* Demands from their own internal security, audit, and compliance teams who expect all internal software to be secure, traceable, up-to-date, and compliant with industry regulations.

## What's the Solution?
This is where Promises can help. The aim of a Promise is simple:
* To enable Platform teams to take complex software, modify the settings needed to meet their internal requirements, inject their own organisational opinions, and finally to expose a simplified API to _their_ users to enable frictionless creation and consumption of services that meet the needs of all stakeholders.

The more Promises a platform can deliver, the richer that platform becomes. While commercial entities will provide high-quality Promises that meet the demands of a broad base of platform teams, it is inevitable any team, in an organisation at scale, will have to respond to requests to add further custom as-a-Service capabilities.

### It's not just about Data Services
We are seeing an increase in demand for Platform teams to ship capabilities that provide higher value than simple, atomic, as-a-Service data services such as Redis or your favourite DbaaS. Data service capabilities are table stakes for a modern platform. Platform users are now demanding increasingly complex patterns of the technologies they need, on-demand, so they can be immediately productive. In other words: they want _their_ required technologies, and they want them wired together, so they can get on with delivering value to _their_ customers! These complex patterns are frequently referred to as [Golden paths](https://engineering.atspotify.com/2020/08/17/how-we-use-golden-paths-to-solve-fragmentation-in-our-software-ecosystem/) or [Paved Paths](https://medium.com/codex/what-is-a-paved-path-b2294463a3a9).

As Promises are the unit that allow platforms to be built incrementally, platform teams can easily add low-level Promises (such as a Jenkins DBaaS, Redis etc.), and then simply wire them together into a single high-level Promise. This raises the "Value Line", reducing the cognitive load for platform users.

Consider a Promise such as 'ACME Java Development Environment': setting up development environments is repetitive and requires many cookie-cutter steps. This Promise can encapsulate the required steps, and handle the toil of wiring up the Git repos, spinning up a CI/CD server, creating a PaaS to run the applications, instructing CI/CD to listen to the Git repos and push successful builds into the PaaS, and finally wiring applications to their required data services. All of this complexity can easily be encapsulated within a single Promise.

### Promise Basics
A Promise is comprised of three elements:

1. `xaasCrd`: this is the CRD that is exposed to the users of the Promise. Imagine the order form for a product. What do you need to know from your customer? Size? Location? Name?
2. `xaasRequestPipeline`: this is the pipeline that will create the Jenkins resources required to run Jenkins on a worker cluster decorated with whatever you need to run Jenkins from your own Platform. Do you need to scan images? Do you need to send a request to an external API for approval? Do you need to inject resources for storage, mesh, networking, etc.? These activities happen in the pipeline.
3. `workerClusterResources`: this contains all of the Kubernetes resources required on a cluster for it to be able to run an instance Jenkins such as CRDs, Operators and Deployments. Think about the required prerequisites necessary on the worker cluster, so that the resources declared by your pipeline are able to converge.

Now that you know more about Kratix Promises, let's install a Kratix Promise locally.

<br>
<hr>
<br>

## <a name="install-jenkins"></a>Quick Start: installing Jenkins as a Kratix Promise

### Prerequisites
* [Install Kratix across two KinD clusters](/installing-kratix/)

### Part 1: Install a Jenkins Promise

For the purpose of this walkthrough let's install the provided Jenkins-as-a-service Kratix Promise.

```
kubectl config use-context kind-platform
kubectl apply -f samples/jenkins/jenkins-promise.yaml
```

On the platform cluster you can now see the ability to create Jenkins instances.

```
kubectl get crds jenkins.example.promise.syntasso.io
```

The above command will give an output similar to:
```
NAME                                     CREATED AT
jenkins.example.promise.syntasso.io   2021-09-03T12:02:20Z
```

On the worker cluster you can see that the Jenkins operator is now installed.

```
kubectl get pods --namespace default --context kind-worker
```

The above command will give an output similar to:
```
NAME                                READY   STATUS    RESTARTS   AGE
jenkins-operator-7886c47f9c-zschr   1/1     Running   0          4m1s
```

Congratulations! You have installed your first Promise. The machinery to issue Jenkins instances on demand by application teams has now been installed.

### Part 2: Request a Jenkins Instance

You will now switch hats and take the role of an Application Developer that wants to request a new instance of Jenkins, using the new platform capability. Requesting the Jenkins is as simple as:

```
kubectl apply -f samples/jenkins/jenkins-resource-request.yaml
```

You can see the request on the platform cluster.

```
kubectl get jenkins.example.promise.syntasso.io
```

The above command will give an output similar to:
```
NAME                   AGE
my-jenkins   27s
```

#### Review created Jenkins instance on the worker cluster

Once Kratix has applied the new configuration to the worker cluster (this will take a few minutes), the Jenkins instance will be created.

```
kubectl get pods --namespace default --context kind-worker
```

The above command will give an output similar to:
```
NAME                                READY   STATUS    RESTARTS   AGE
jenkins-example                     1/1     Running   0          113s
jenkins-operator-7886c47f9c-zschr   1/1     Running   0          19m
```

#### Using your Jenkins instance

You can access the Jenkins UI in a browser. For that, you need the credentials:
1. Get the Jenkins username: `kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d`
2. Get the Jenkins password: `kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d`
3. `kubectl --context kind-worker port-forward jenkins-example 8080:8080`
4. Navigate to http://localhost:8080 and login using the username and password captured in steps one and two.
5. You should see a Seed Job in the Jenkins UI, and a corresponding Pod on your Worker cluster.

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have installed a Kratix Promise and used it to create on-demand instances of a service. <br/>
👉🏾&nbsp;&nbsp; Let's [deploy a web app that uses multiple Kratix Promises](/using-multiple-promises/README.md).