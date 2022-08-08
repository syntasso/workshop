This is Part 2 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Quick Start: Install Kratix](/installing-kratix/) <br/>
👉🏾&nbsp;&nbsp; Next: [Using multiple Kratix Promises](/using-multiple-promises/)

<hr>

### In this tutorial, you will
1. [learn more about Kratix Promises](#promise)
1. [install Jenkins as a Kratix Promise](#install-jenkins)

# <a name="promise"></a> What is a Kratix Promise?

If you are, or have been, a member of a platform team, you'll know how hard it can be. We've been platform team members, we've worked with many platform teams, and we've consistently experienced shared pains such as:

* Work is hard, but for the wrong reasons.
* Work involves managing tension from many teams, parts of the business, and key stakeholders.
* Customers or users expect software served from the platform to be as simple, quick to consume, and performant as commodity public-cloud services.

Kratix and Promises _exist_ to help platform teams to minimise these pains and deliver value more easily.

We described the Kratix framework in the [previous step](/installing-kratix/README.md), now we want to talk through the high-level capabilities and anatomy of a Kratix Promise.

Conceptually, Promises are the building blocks of Kratix that allow you to develop your platform incrementally. Technically, a Promise is a YAML document that defines a contract between the Platform and its users. We will explore more about the internals of a Kratix Promise in part 4 where you will [write your own Promise](/writing-a-promise/README.md).

## Kratix Promises

* enable you to build your platform incrementally and in response to the needs of your users.
* codify the contract between platform teams and application teams for the delivery of a specific service, e.g. a database, an identity service, a supply chain, or a complete development pipeline of patterns and tools.
* are easy to build, deploy, and update.
* are sharable and reusable between platforms, teams, business units, and other organisations.
* add up to a frictionless experience when platform users want to create services that they need to deliver value.

Now that you know more about Kratix Promises, follow the steps below to install a Promise.

<br>
<hr>
<br>

## <a name="install-jenkins"></a>Installing Jenkins as a Kratix Promise

### Steps
1. [Complete pre-requistes](#prerequisites), if required
1. [Install the Jenkins Promise](#install-promise)
1. [Request an instance](#request-instance)
1. [Use the instance](#use-instance)
1. [Tear down your environment](#teardown)


### <a name="prerequisites">Prerequisites
* [Install Kratix across two KinD clusters](/installing-kratix/)

### <a name="install-promise"> Install the off-the-shelf Jenkins Promise

Install the provided Jenkins-as-a-service Kratix Promise.

```console
kubectl config use-context kind-platform
kubectl apply -f samples/jenkins/jenkins-promise.yaml
```

Verify you know have the ability to create Jenkins instances.
```console
kubectl get crds jenkins.example.promise.syntasso.io
```

The above command will give an output similar to
```console
NAME                                     CREATED AT
jenkins.example.promise.syntasso.io   2021-09-03T12:02:20Z
```

Verify that the Jenkins operator is now installed.

```console
kubectl get pods --namespace default --context kind-worker
```

The above command will give an output similar to:
```
NAME                                READY   STATUS    RESTARTS   AGE
jenkins-operator-7886c47f9c-zschr   1/1     Running   0          4m1s
```

Congratulations! You have installed your first Promise. The machinery to issue Jenkins instances on demand by application teams has now been installed.

### <a name="request-instance">Request a Jenkins Instance

Submit a _resource request_ to get an instance of Jenkins.
```console
kubectl apply -f samples/jenkins/jenkins-resource-request.yaml
```

Verify that the _resource request_ was issued on the platform cluster.

```console
kubectl get jenkins.example.promise.syntasso.io
```

The above command will give an output similar to
```console
NAME                   AGE
my-jenkins   27s
```

Verify the instance is created on the worker cluster (this may take a few minutes so `-w` will watch the output).
```console
kubectl get pods --namespace default --context kind-worker -w
```

The above command will give an output similar to
```console
NAME                                READY   STATUS    RESTARTS   AGE
jenkins-example                     1/1     Running   0          113s
jenkins-operator-7886c47f9c-zschr   1/1     Running   0          19m
```

### <a name="use-instance">Use your Jenkins instance

You can access the Jenkins UI in a browser. 

Port forward for browser access to the Jenkins UI 
```console
kubectl --context kind-worker port-forward jenkins-example 8080:8080
```
<br>

Navigate to http://localhost:8080 and login with the credentials you copy from the below commands.

<br>

Copy and paste the Jenkins username into the login page
```console
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d | pbcopy
```
<br>

Copy and paste the Jenkins password into the login page
```console
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d | pbcopy
```
<br>

Verify there is a Seed Job in the Jenkins UI and a corresponding Pod on your Worker cluster

### <a name="teardown">Tearing it all down

The next section in this tutorial requires a clean Kratix installation. Before heading to it, please clean up your environment by running:

```console
kind delete clusters platform worker
```
<br>

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have installed a Kratix Promise and used it to create on-demand instances of a service. <br/>
👉🏾&nbsp;&nbsp; Next you will [deploy a web app that uses multiple Kratix Promises](/using-multiple-promises/README.md).
