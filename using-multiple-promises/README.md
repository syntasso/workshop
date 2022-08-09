This is Part 3 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Quick Start: Install a Kratix Promise](/installing-a-promise/) <br/>
👉🏾&nbsp;&nbsp; Next: [Writing and installing a Kratix Promise](/writing-a-promise/)

<hr>

### In this tutorial, you will
1. [learn more about the power of Promises](#power-of-promises)
1. [use Kratix Promises to build a Golden Path](#deploy)

# <a name="power-of-promises"></a> The power of Promises

As covered in previously, Promises are the blocks that enable teams to build platforms that specifically meet their customer needs. Through writing and extending Promises, Platform teams can raise the value line of the platform they provide. They can wire together simpler, low-level Promises to provide a single, unified high-level Promise, tailored to their users needs.

Consider the task of setting up development environments for application teams. This task is usually repetitive and requires many cookie-cutter steps. It may involve wiring up Git repos, spinning up a CI/CD server, creating a PaaS to run the applications, instructing CI/CD to listen to the Git repos and push successful builds into the PaaS, and finally wiring applications to their required data services.

A Promise can encapsulate all the required steps and handle the toil of running those low-level tasks. It can be designed as a single Promise that does it all, or it can be a collection of Promises that, combined, deliver the desired functionality. All your users do is request a new environment. Behind the scenes, multiple Promises are working together, each with their own responsibility.

Now you will see the power of Kratix Promises by deploying a web app that uses multiple Promises.

<br>
<hr>
<br>

## <a name="deploy"></a> Building a Golden Path using multiple Kratix Promises

### Steps
1. [Complete pre-requistes](#prerequisites), if required
1. [Install Promises](#install-all-promises)
1. [Request instances](#request-instances)
1. [Run the deploy pipeline](#deploy-pipeline)
1. [Test the application](#test-app)
1. [Tear down your environment](#teardown)

### <a name="prerequisites"></a>Pre-requisites
* [Install Kratix across two KinD clusters](/installing-kratix/)

### <a name="install-all-promises">Install all required Promises

Install the required Promises on your Platform cluster:

<!-- ❓ Do we want people to clone the workshop and kratix or not? -->
```console
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/postgres/postgres-promise.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/knative-serving/knative-serving-promise.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/jenkins/jenkins-promise.yaml
```
<br>

Verify the Promises are all installed on your Platform cluster
```console
kubectl --context kind-platform get promises
```
<br>

The above command will give an output similar to
```console
NAME                      AGE
ha-postgres-promise       1h
jenkins-promise           1h
knative-serving-promise   1h
```
<br>

### <a name="request-instances">Request instances

Submit a _resource request_ to get a Knative Serving component, a Jenkins instance and a Postgres database.
```console
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/postgres/postgres-resource-request.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/knative-serving/knative-serving-resource-request.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/jenkins/jenkins-resource-request.yaml
```
<br>

Verify you have all the necessary resources up and running (this may take a few minutes so `-w` will watch the command).
```console
kubectl --context kind-worker get pods -w
```
<br>

The above command will give an output similar to
```console
NAME                                      READY   STATUS    RESTARTS         AGE
acid-minimal-cluster-0                    1/1     Running   0                3m10s
acid-minimal-cluster-1                    1/1     Running   0                2m43s
jenkins-example                           1/1     Running   0                76m
...
```
<br>

and

<br>

```console
kubectl --context kind-worker get namespaces -w
```
<br>

The above command will give an output similar to
```console
NAME                   STATUS   AGE
knative-serving        Active   1h
kourier-system         Active   1h
...
```
<br>

Verify that the _resource request_ was issued on the platform cluster.
```console
kubectl get jenkins.example.promise.syntasso.io
```
<br>

The above command will give an output similar to
```console
NAME          AGE
my-jenkins    27s
```
<br>

Verify the instance is created on the worker cluster (this may take a few minutes so `-w` will watch the output).
```console
kubectl get pods --namespace default --context kind-worker -w
```
<br>

The above command will give an output similar to
```console
NAME                                READY   STATUS    RESTARTS   AGE
jenkins-example                     1/1     Running   0          113s
jenkins-operator-7886c47f9c-zschr   1/1     Running   0          19m
```
<br>

#### <a name="deploy-pipeline">Run the deploy pipeline

Access the Jenkins UI in a browser, as in the [previous step](/installing-a-promise/README.md).

<br>

Port forward for browser access to the Jenkins UI 
```console
kubectl --context kind-worker port-forward jenkins-example 8080:8080
```
<br>

Navigate to http://localhost:8080 and log in with the credentials you copy from the below commands.

<br>

Copy and paste the Jenkins username into the login page
```console
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d
```
<br>

Copy and paste the Jenkins password into the login page
```console
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d
```
<br>

In the Jenkins UI, create a new pipeline using this
[Jenkinsfile](https://raw.githubusercontent.com/syntasso/workshop/main/sample-todo-app/ci/Jenkinsfile)
and execute it.

https://user-images.githubusercontent.com/201163/175933452-853af525-7fff-4dca-9ba9-032c07c8c393.mov

<br>

### <a name="validate-deployment">Validate the deployment

Verify that the Knative Service for the application is ready:

```console
kubectl --context kind-worker get services.serving.knative.dev
```
<br>

The above command will give an output similar to
```console
NAME   URL                               LATESTCREATED   LATESTREADY   READY   REASON
todo   http://todo.default.example.com   todo-00001      todo-00001    True
```
<br>

### <a name="test-app">Test the deployed application

Now test the app. 

On a separate terminal, you'll need to open access to the app by port-forwarding the kourier service:

```console
kubectl --context kind-worker --namespace kourier-system port-forward svc/kourier 8081:80
```
<br>

Now curl the app:
```console
curl -H "Host: todo.default.example.com" localhost:8081
```
<br>

### <a name="teardown">Tearing it all down

The next section in this tutorial requires a clean Kratix installation. Before heading to it, please clean up your environment by running:

```console
kind delete clusters platform worker
```
<br>

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have deployed a web app that uses multiple Kratix Promises. <br/>
👉🏾&nbsp;&nbsp; Now you will [write your own Jenkins Promise to learn more about how Kratix Promises work](/writing-a-promise/README.md).
