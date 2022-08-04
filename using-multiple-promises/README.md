This is Part 3 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Quick Start: Install a Kratix Promise](/installing-a-promise/) <br/>
👉🏾&nbsp;&nbsp; Next: [Writing and installing a Kratix Promise](/writing-a-promise/)

<hr>

### In this tutorial, you will
1. [learn more about the power of Promises](#power-of-promises)
1. [deploy a web app that uses multiple Kratix Promises](#deploy)

# <a name="power-of-promises"></a> The power of Promises

As covered in previously, Promises are the blocks that enable teams to build platforms that specifically meet their customer needs. Through writing and extending Promises, Platform teams can raise the value line of the platform they provide. They can wire together simpler, low-level Promises to provide a single, unified high-level Promise, tailored to their users needs.

Consider the task of setting up development environments for application teams. This task is usually repetitive and requires many cookie-cutter steps. It may involve wiring up Git repos, spinning up a CI/CD server, creating a PaaS to run the applications, instructing CI/CD to listen to the Git repos and push successful builds into the PaaS, and finally wiring applications to their required data services.

A Promise can encapsulate all the required steps and handle the toil of running those low-level tasks. It can be designed as a single Promise that does it all, or it can be a collection of Promises that, combined, deliver the desired functionality. All your users do is request a new environment. Behind the scenes, multiple Promises are working together, each with their own responsibility.

Let's demonstrate the power of Kratix Promises by deploying a web app that uses multiple Promises.

<br>
<hr>
<br>

## <a name="deploy"></a> Deploying a web app using multiple Kratix Promises

This sample application workflow shows how to combine Kratix Promises
to deploy a web app.

### Pre-requisites
* [Install Kratix across two KinD clusters](/installing-kratix/)

### Install all required Promises

At this stage, you should have Kratix installed across two clusters:

```console
$ kubectl config get-clusters
NAME
kind-worker
kind-platform

$ kubectl --context kind-platform get namespace kratix-platform-system
NAME                     STATUS   AGE
kratix-platform-system   Active   1h

$ kubectl --context kind-worker get namespace kratix-worker-system
NAME                   STATUS   AGE
kratix-worker-system   Active   1h
```

You can now install the required Promises on your Platform cluster:

<!-- ❓ Do we want people to clone the workshop and kratix or not? -->
```bash
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/postgres/postgres-promise.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/knative-serving/knative-serving-promise.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/jenkins/jenkins-promise.yaml
```

### Request all the resources

At this stage, the required Promises are all installed on your Platform cluster:

```console
$ kubectl --context kind-platform get promises
NAME                      AGE
ha-postgres-promise       1h
jenkins-promise           1h
knative-serving-promise   1h
```

You can now request a Knative Serving component, a Jenkins instance and a Postgres
database:

```bash
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/postgres/postgres-resource-request.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/knative-serving/knative-serving-resource-request.yaml
kubectl --context kind-platform apply --filename https://raw.githubusercontent.com/syntasso/kratix/main/samples/jenkins/jenkins-resource-request.yaml
```

### Deploy the app using Jenkins

Yoo should, after a few minutes, have all the necessary resources up and running:

```console
$ kubectl --context kind-worker get pods
NAME                                      READY   STATUS    RESTARTS         AGE
acid-minimal-cluster-0                    1/1     Running   0                3m10s
acid-minimal-cluster-1                    1/1     Running   0                2m43s
jenkins-example                           1/1     Running   0                76m
...

$ kubectl --context kind-worker get namespaces
NAME                   STATUS   AGE
knative-serving        Active   1h
kourier-system         Active   1h
...
```

### Verify your Jenkins installation

You can verify your Jenkins to ensure it's up and running. First, you'll need to open access to the instance by running the following command on a dedicated terminal:

```bash
kubectl --context kind-worker port-forward pod/jenkins-example 8080:8080
```

You can now navigate to http://localhost:8080 and login. The username is
`jenkins-operator` and the password can be found by running:

```bash
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d
```
#### Run the deploy pipeline

Create a new pipeline using this
[Jenkinsfile](https://raw.githubusercontent.com/syntasso/workshop/main/sample-todo-app/ci/Jenkinsfile)
and execute it.

https://user-images.githubusercontent.com/201163/175933452-853af525-7fff-4dca-9ba9-032c07c8c393.mov

### Validate the deployment

At this stage, the Knative Service for the application is ready:

```console
$ kubectl --context kind-worker get services.serving.knative.dev
NAME   URL                               LATESTCREATED   LATESTREADY   READY   REASON
todo   http://todo.default.example.com   todo-00001      todo-00001    True
```

You can now test the app. On a separate terminal, you'll need to open access to
the app by port-forwarding the kourier service:

```bash
kubectl --context kind-worker --namespace kourier-system port-forward svc/kourier 8081:80
```

You can now curl the app:

```
curl -H "Host: todo.default.example.com" localhost:8081
```

### Tearing it all down

The next section in this tutorial requires a clean Kratix installation. Before heading to it, please clean up your environment by running:

```bash
kind delete clusters platform worker
```

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have deployed a web app that uses multiple Kratix Promises. <br/>
👉🏾&nbsp;&nbsp; Let's [write our own Jenkins Promise to learn more about how Kratix Promises work](/writing-a-promise/README.md).
