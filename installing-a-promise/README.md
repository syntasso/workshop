This is Part 2 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Quick Start: Install Kratix](/installing-kratix/) <br/>
👉🏾&nbsp;&nbsp; Next: [Using multiple Kratix Promises](/using-multiple-promises/)

<hr>

### In this tutorial, you will
1. [learn more about Kratix Promises](#promise)
1. [install Jenkins as a Kratix Promise](#install-jenkins)

# <a name="promise"></a> What is a Kratix Promise?

We've been platform team members. We've worked with many platform team members. We've talked to many platform team members.

We consistently share pains in this role:

* Work can be hard, but for the wrong reasons.

* Work involves managing tension from many teams, parts of the business, and key stakeholders.

* Customers or consumers expect software served from the platform to be as simple, quick to consume, and performant as commodity public-cloud services.

Kratix and Promises _only exist_ to help platform teams to minimise these pains and deliver value more easily.

We spoke about Kratix as the framework in the [previous step](/installing-kratix/README.md), now we want to talk through the high-level capabilities and anatomy of the Kratix Promise.

## Kratix Promises

* provide the right abstractions to make your developers as productive, efficient, and secure as possible. Any capability can be encoded and delivered via a Promise, and once “Promised” the capability is available on-demand, at scale, across the organisation.

* codify the contract between platform teams and application teams for the delivery of a specific service, e.g. a database, an identity service, a supply chain, or a complete development pipeline of patterns and tools.

* can be shared and reused between platforms, teams, business units, even other organisations.

* are easy to build, deploy, and update. Bespoke business logic can be added to each Promise’s pipeline.

## Kratix Promise anatomy

A Promise is comprised of three elements:
* **Custom Resource Definition**: input from an app team to create instances of a capability.

* **Worker Cluster Resources**: dependencies necessary for any created Workloads.

* **Request Pipeline**: business logic required when an instance of a capability is requested.

We will explore more about the internals of a Kratix Promise in part 4 where we [write our own Promise](/writing-a-promise/README.md). For now, let's install an off-the-shelf Kratix Promise locally.

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

### Tearing it all down

The next section in this tutorial requires a clean Kratix installation. Before heading to it, please clean up your environment by running:

```bash
kind delete clusters platform worker
```

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have installed a Kratix Promise and used it to create on-demand instances of a service. <br/>
👉🏾&nbsp;&nbsp; Let's [deploy a web app that uses multiple Kratix Promises](/using-multiple-promises/README.md).
