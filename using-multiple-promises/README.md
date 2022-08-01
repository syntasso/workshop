# Deploying a web app using multiple Kratix Promises

This sample application workflow shows how to combine Kratix Promises
to deploy a web app.

If you wish to run this app locally, please check the necessary
environment variables 
[here](https://github.com/syntasso/workshop/blob/fd5188b89164da9be70e664d1048d897dcf202f0/sample-todo-app/main.go#L21-L25).

## Pre-requisite 
* [Install Kratix across two KinD clusters](/installing-kratix/)

## Install all required Promises

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

## Request all the resources

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

## Deploy the app using Jenkins

You should now have all the necessary resources up and running:

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

### Startup Jenkins

Jenkins runs a few tasks on first login, so let's get that started. First,
you'll need to open access to the instance by running the following command on a
dedicated terminal:

```bash
kubectl --context kind-worker port-forward pod/jenkins-example 8080:8080
```

You can now navigate to http://localhost:8080 and login. The username is
`jenkins-operator` and the password can be found by running:

```bash
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d
```

### Create a Service Account

<!-- This could later be added to the existing jenkins Promise to simplify this step  -->

To deploy the app on the worker cluster, you need to set up a Service Account
with the correct permissions:

```bash
kubectl --context kind-worker apply --filename https://raw.githubusercontent.com/syntasso/workshop/main/sample-todo-app/k8s/deploy-rbac.yaml
```

### Run the deploy pipeline

For the pipeline to work, you'll need the following Service Account:

```console
$ kubectl --context kind-worker get serviceaccounts
NAME                       SECRETS   AGE
knative-jenkins-deployer   0         2s
...
```

Create a new pipeline using this
[Jenkinsfile](https://raw.githubusercontent.com/syntasso/workshop/main/sample-todo-app/ci/Jenkinsfile)
and execute it.

https://user-images.githubusercontent.com/201163/175933452-853af525-7fff-4dca-9ba9-032c07c8c393.mov

## Validate the deployment

At this stage, the Knative Service for the application is ready:

```console
$ kubectl --context kind-worker get services.serving.knative.dev
NAME   URL                               LATESTCREATED   LATESTREADY   READY   REASON
todo   http://todo.default.example.com   todo-00001      todo-00001    True
```

We can now test the app. On a separate terminal, you'll need to open access to
the app by port-forwarding the kourier service:

```bash
kubectl --context kind-worker --namespace kourier-system port-forward svc/kourier 8081:80
```

You can now curl the app:

```
curl -H "Host: todo.default.example.com" localhost:8081
```