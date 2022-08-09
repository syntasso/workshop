This is Part 4 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Using multiple Kratix Promises](/using-multiple-promises/) <br/>
👉🏾&nbsp;&nbsp; Next: [Enhancing a Kratix Promise](/enhancing-a-promise/)

<hr>

### In this tutorial, you will
1. [learn more about what's inside a Kratix Promise](#inside-a-promise)
1. [write and install your own Kratix Promise](#write-promise-start)

# <a name="inside-a-promise">What's inside a Kratix Promise?

You've [installed Kratix and three off-the-shelf Promises](/using-multiple-promises/README.md). Now you'll create a Promise from scratch.

From [installing a Promise](/installing-a-promise/README.md), a Kratix Promise is a YAML document that defines a contract between the platform and its users. It is what allows platforms to be built incrementally. 

A Promise consists of three parts:
1. `xaasCrd`: the CRD that is exposed to the users of the Promise.
1. `workerClusterResources`: a collection of Kubernetes resources to be installed in the worker clusters.
1. `xaasRequestPipeline`: the pipeline that will create the resources requried to run an instance of the promised service on a worker cluster.

## Promise parts, in detail

### `xaasCrd`
The `xaasCrd` is your user-facing API for the Promise. It defines the options that users can configure when they request the Promise. The complexity of the `xaasCrd` API is up to you. You can read more about writing Custom Resource Definitions in the [Kubernetes docs](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#create-a-customresourcedefinition).

### `workerClusterResources`
The `workerClusterResources` describes everything required to fulfil the Promise. Kratix applies this content on all registered worker clusters. For instance with the Jenkins Promise, the `workerClusterResources` contains the Jenkins CRD, the Jenkins Operator, and the resources the Operator requires. 

### `xaasRequestPipeline`
The `xaasRequestPipeline` defines a set of jobs to run when Kratix receives a request for an instance of one of its Promises. 

The pipeline is an array of Docker images, and those images are executed in order. The pipeline enables you to write Promises with specialised images and combine those images as needed.

#### How the pipeline works

Each container in the `xaasRequestPipeline` array will output complete, valid Kubernetes resources. 

The contract with each pipeline container is simple and straightforward:
* The first container in the list receives the resource document created by the user's request&mdash;this request will comply with the `xaasCrd` described above. The document is available to the pipeline in `/input/object.yaml`.
* The container's command then executes with the input object and fulfils its responsibilites.
* The container writes any resources to be created to `/output/`.
* The resources in `/output` of the last container in the `xaasRequestPipeline` array will be scheduled and applied to the appropriate worker clusters.

## Recap: basics of getting a promised instance to your users

At a very high level

* You talk to users of your platform to find out what they're using and what they need.
* You write a Kratix Promise for a service that your users and teams need.
  * In `xaasCrd`, you list what your users can configure in their request.
  * In `workerClusterResources`, you list what resources are required for Kratix to fulfil the Promise.
  * In `xaasRequestPipeline`, you list Docker images that will take the user's request and decorate it with configuration that you or the business require.
* You install the Promise on your platform cluster, where Kratix is installed. 
* Your user wants an instance of the Promise.
* Your user submit what Kratix calls a _resource request_ that lists what they want and how they want it, and this complies with the `xaasCrd` (more details on this request later).
* Kratix fires off the request pipeline that you defined in `xaasRequestPipeline` and passes the _resource request_ as an input.
* The pipeline outputs valid Kubernetes documents that say what the user wants and what the business wants for that Promise instance.
* The worker cluster has what it needs based on the `workerClusterResources` and is ready to create the instance when the request comes through.

## A Jenkins Promise

Imagine your platform team has received its fourth request from its fourth team for a Jenkins instance. You decide four times is too many times to manually set up Jenkins. 

Now you'll write a Jenkins Promise and install it on your platform so that your four teams get Jenkins&mdash;and you get time back for more valuable work. 

<br>
<hr>
<br>


## <a name="write-promise-start"> Writing your own Kratix Promise

### Steps
1. [Complete pre-requistes](#prerequisites), if required
1. [Folder setup](#folder-setup)
1. [Create a Promise template](#promise-template)
1. [X-as-a-Service Custom Resource Definition: define your Promise API](#define-crd)
1. [Create your Promise instance base manifest](#base-instance)
1. [Create simple request pipeline functionality](#pipeline-script)
1. [Define your Docker image for the pipeline](#dockerfile)
1. [Test your container image](#test-image)
1. [Create your Promise definition and define your `workerClusterResources`](#worker-cluster-resources)
1. [Install your Promise](#install-promise)
1. [Create and submit a resource request](#create-resource-request)
1. [Summary](#summary)
1. [Tear down your environment](#teardown)


### <a name="prerequisites"></a>Pre-requisites

You need a fresh installation of Kratix for this section. The simplest way
to do so is by running the quick-start script from within the Kratix
directory:

```bash
cd /path/to/kratix
./scripts/quick-start --recreate
```

Alternatively, you can go back to the first step on this series: [Install Kratix across two KinD clusters](/installing-kratix/).


### <a name="folder-setup">Folder setup

To begin writing a Promise you will need a basic directory structure to work in:

```console
mkdir -p jenkins-promise/{resources,request-pipeline-image}
cd jenkins-promise
```

### <a name="promise-template">Create a Promise template

Create a basic `jenkins-promise-template.yaml` to work with
```console
cat > jenkins-promise-template.yaml <<EOF
apiVersion: platform.kratix.io/v1alpha1
kind: Promise
metadata:
  name: jenkins-promise
spec:
  #injected via: go run path/to/kratix/hack/worker-resource-builder/main.go -k8s-resources-directory ${PWD}/resources -promise ${PWD}/jenkins-promise-template.yaml > jenkins-promise.yaml
  #workerClusterResources:
  xaasRequestPipeline:
  xaasCrd:
EOF
```
<br>
You will fill the `spec` scalars as you progress through the tutorial.

### <a name="define-crd">X-as-a-Service Custom Resource Definition: define your Promise API
For the purpose of this tutorial, you will create an API that accepts a single `string` parameter called `name`. This API can be as complex or as simple as you design it to be.

Add the below to the `xaasCrd` scalar in `jenkins-promise-template.yaml`. Ensure the indentation is correct (`xaasCrd` is nested under `spec`).

```yaml
  xaasCrd:
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
      name: jenkins.promise.example.com
    spec:
      group: promise.example.com
      scope: Namespaced
      names:
        plural: jenkins
        singular: jenkins
        kind: jenkins
      versions:
      - name: v1
        served: true
        storage: true
        schema:
          openAPIV3Schema:
            type: object
            properties:
              spec:
                type: object
                properties:
                  name:
                    type: string
```

You have now created the as-a-Service API.

### <a name="base-instance">Create your Promise instance base manifest

Next build the pipeline to transform a Promise _resource request_ into the Kubernetes resources required to create a running instance of the Jenkins service.

```console
cd request-pipeline-image
```

Create the <code>jenkins-instance.yaml</code> base manifest file that the pipeline will transform.

<details>
  <summary>👀&nbsp;&nbsp;<strong>CLICK HERE</strong> to expand the code you need to create the <code>jenkins-instance.yaml</code> file.</summary>

```console
cat > jenkins-instance.yaml <<EOF
apiVersion: jenkins.io/v1alpha2
kind: Jenkins
metadata:
  name: <tbr-name>
  namespace: default
spec:
  configurationAsCode:
    configurations: []
    secret:
      name: ""
  groovyScripts:
    configurations: []
    secret:
      name: ""
  jenkinsAPISettings:
    authorizationStrategy: createUser
  master:
    basePlugins:
    - name: kubernetes
      version: "1.31.3"
    - name: workflow-job
      version: "1180.v04c4e75dce43"
    - name: workflow-aggregator
      version: "2.7"
    - name: git
      version: "4.11.0"
    - name: job-dsl
      version: "1.79"
    - name: configuration-as-code
      version: "1414.v878271fc496f"
    - name: kubernetes-credentials-provider
      version: "0.20"
    disableCSRFProtection: false
    containers:
      - name: jenkins-master
        image: jenkins/jenkins:2.332.2-lts
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 12
          httpGet:
            path: /login
            port: http
            scheme: HTTP
          initialDelaySeconds: 100
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        readinessProbe:
          failureThreshold: 10
          httpGet:
            path: /login
            port: http
            scheme: HTTP
          initialDelaySeconds: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 1500m
            memory: 3Gi
          requests:
            cpu: "1"
            memory: 500Mi
  seedJobs:
    - id: jenkins-operator
      targets: "cicd/jobs/*.jenkins"
      description: "Jenkins Operator repository"
      repositoryBranch: master
      repositoryUrl: https://github.com/jenkinsci/kubernetes-operator.git
EOF
```
</details>

### <a name="pipeline-script"> Create simple request pipeline functionality

Kratix takes no opinion on the tooling used within a pipeline. Kratix will pass a set of resources to the pipeline, and expect back a set of resources. What happens within the pipeline, and what tooling is used, is a decision left entirely to you. 

For this example, you're taking a name from the _resource request_ for an instance and passing it to the Jenkins custom resource output. 

To keep this transformation simple, you'll use a combination of `sed` and `yq` to do the work.

Create a script file that will execute when the pipeline runs.
```console
cat > execute-pipeline.sh <<EOF
#!/bin/sh
#Get the name from the Promise Custom resource
instanceName=\$(yq eval '.spec.name' /input/object.yaml)

# Inject the name into the Jenkins resources
find /tmp/transfer -type f -exec sed -i \\
  -e "s/<tbr-name>/\${instanceName//\//\\/}/g" \\
  {} \;

cp /tmp/transfer/* /output/
EOF
```
<br>

Then make it executable:
```console
chmod +x execute-pipeline.sh
```
<br>

### <a name="dockerfile"> Define your Docker image for the pipeline

Run the code below, which creates a `Dockerfile` that will 
* copy the `jenkins-instance.yaml` doc you created above into Kratix
* transform the file by running the `execute-pipeline.sh` script you created above
* pass the transformed file to the `worker` cluster ready for execution

```console
cat > Dockerfile <<EOF
FROM "mikefarah/yq:4"
RUN [ "mkdir", "/tmp/transfer" ]

ADD jenkins-instance.yaml /tmp/transfer/jenkins-instance.yaml
ADD execute-pipeline.sh execute-pipeline.sh

CMD [ "sh", "-c", "./execute-pipeline.sh"]
ENTRYPOINT []
EOF
```
<br>

Next build your Docker image
```console
docker build . --tag <your-org-name/name-of-your-image>
```

### <a name="test-image">Test your Docker container image

Test the Docker container image by supplying an input resource and examining the output resource.

Create the test input and output directories locally.

```console
mkdir {input,output}
```

The `/input` directory is where your incoming _resource request_ will be written when a user wants an instance.

Create a sample user resource request to the `/input` directory
```console
cat > input/object.yaml <<EOF
apiVersion: promise.example.com/v1
kind: jenkins
metadata:
  name: my-jenkins-promise-request
spec:
  name: my-amazing-jenkins
EOF
```
<br>

Run the container and examine the output
```console
docker run -v ${PWD}/input:/input -v ${PWD}/output:/output <your-org-name/name-of-your-image>
cat output/*
```
<br>

The contents of the `/output` directory will be scheduled and deployed by Kratix to a worker cluster. They need to be valid Kubernetes resources that can be applied to any cluster with the Promise's `workerClusterResources` installed (see beneath).

Once you are satisified with the image, push it so it's ready for use.
```console
docker push <your-org-name/name-of-your-image>
```
<br>

The final step of creating the `xaasRequestPipeline` is to add the  `spec.xaasRequestPipeline` scalar in `jenkins-promise-template.yaml`.

Go back to the `jenkins-promise` directory.

```console
cd ..
```

Add the image to the array in `jenkins-promise-template.yaml`.
```yaml
xaasRequestPipeline:
    - <your-org-name/name-of-your-image>
```
<br>

In summary, you have:
- Created a container image containing:
    - A template file to be injected with per-instance details (`jenkins-instance.yaml`)
    - A shell script to retrieve the per-instance details from the user's request, and inject them into the template (`execute-pipeline.sh`)
    - A command set to the shell script
- Created a set of directories(`input`/`output`) and sample user request(`input/object.yaml`)
- Executed the pipeline image locally as a test
- Pushed the image to the registry
- Added the image to the Promise definition in the `xaasRequestPipeline` array


### <a name="worker-cluster-resources">Create your Promise definition and define your `workerClusterResources`

The `workerClusterResources` describes everything required to fulfil the Promise. Kratix applies this content on all registered worker clusters. 

For this promise, the `workerClusterResources` needs to contain the Jenkins CRD, the Jenkins Operator, and the resources the Operator requires. 

 Jenkins.io has a [great Operator](https://jenkinsci.github.io/kubernetes-operator/docs/getting-started/latest/installing-the-operator/) that ships in two files.
1. [Jenkins CRDs](https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/config/crd/bases/jenkins.io_jenkins.yaml)
2. [The Operator](https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/deploy/all-in-one-v1alpha2.yaml) and other required resources such as Service Accounts, Role Bindings and Deployments.

Download both.

```console
wget https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/fbea1ed790e7a9deb2311e1f565ee93f07d89022/config/crd/bases/jenkins.io_jenkins.yaml -P resources
wget https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/8fee7f2806c363a5ceae569a725c17ef82ff2b58/deploy/all-in-one-v1alpha2.yaml -P resources
```
<br>

Next inject Jenkins files into the `jenkins-promise-template.yaml`. 

To make this step simpler we have written a _very basic_ tool to grab all YAML documents from all YAML files located in `resources` and inject them into the `workerClusterResources` scalar.

```console
go run path/to/kratix/hack/worker-resource-builder/main.go \
  -k8s-resources-directory ${PWD}/resources \
  -promise ${PWD}/jenkins-promise-template.yaml > jenkins-promise.yaml
```
<br>

This created your finished Promise defition, `jenkins-promise.yaml`.


### <a name="install-promise">Install your Promise

Install the Promise in Kratix.

```
kubectl apply --context kind-platform -f jenkins-promise.yaml
```
<br>

Verify the Promise installed (this may take a few minutes so `-w` will watch the command)
```console
kubectl --context kind-platform get crds -w
```
<br>

The above command will give an output similar to
```console
NAME                          CREATED AT
jenkins.promise.example.com   2021-09-09T11:21:10Z
```
<br>

Verify the Jenkins Operator is running (this will take a few minutes so `-w` will watch the command)
```console
kubectl --context=kind-worker get pods -A -w
```
<br>

### <a name="create-resource-request">Create and submit a resource request

You can now be able to request instances of Jenkins.
```console
cat > jenkins-resource-request.yaml <<EOF
apiVersion: promise.example.com/v1
kind: jenkins
metadata:
  name: my-jenkins-promise-request
spec:
  name: my-amazing-jenkins
EOF

kubectl apply --context kind-platform -f jenkins-resource-request.yaml
```
<br>

After a few minutes the Jenkins operator will have received the request and asked the worker to start an instance of Jenkins. 

Target the worker cluster to see the Jenkins instance
```console
kubectl config use-context kind-worker
kubectl get pods -A -w
```
<br>

The above command will give an output similar to
```console
NAME                                READY   STATUS    RESTARTS   AGE
jenkins-my-amazing-jenkins          1/1     Running   0          113s
...
```
<br>

For verification, access the Jenkins UI in a browser, as in [previous steps](/installing-a-promise/README.md).

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
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d | pbcopy
```
<br>

Copy and paste the Jenkins password into the login page
```console
kubectl --context kind-worker get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d | pbcopy
```
<br>

### <a name="teardown">Tearing it all down

The next section in this tutorial requires a clean Kratix installation. Before heading to it, please clean up your environment by running:

```console
kind delete clusters platform worker
```
<br>

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have written a Kratix Promise. <br/>
👉🏾&nbsp;&nbsp; Let's [see how to tailor Kratix Promises based on organisational context](/enhancing-a-promise/README.md).
