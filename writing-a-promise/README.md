This is Part 4 of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Using multiple Kratix Promises](/using-multiple-promises/) <br/>
👉🏾&nbsp;&nbsp; Next: [Enhancing a Kratix Promise](/enhancing-a-promise/)

<hr>

### In this tutorial, you will
1. [learn more about what's inside a Kratix Promise](https://github.com/syntasso/workshop/tree/main/writing-a-promise/README.md#whats-inside-a-kratix-promise)
1. [write and install your own Kratix Promise](https://github.com/syntasso/workshop/tree/main/writing-a-promise/README.md#writing-your-own-kratix-promise)

# What's inside a Kratix Promise?

Now that you know more about how Kratix works, let's write and deploy our own Kratix Promise. First, let's cover the basics of what a Promise actually is, and dive deeper into its components.

## What's a Promise?

In a nutshell, a Promise is a YAML document that defines a contract between the Platform and its users. It is the piece that allows platforms to be built incrementally. They enable Platform teams to take complex software, modify the settings needed to meet their internal requirements, inject their own organisational opinions, and to expose a simplified API to their users to enable frictionless creation and consumption of services that meet the needs of all stakeholders.

By writing and extending Promises, Platform teams can raise the value line of the platform they provide. They can wire together simpler, low-level Promises to provide a single, unified high-level Promise, more customised to their users needs.

Consider the task of setting up development environments. This task is usually repetitive and requires many cookie-cutter steps. A Promise can encapsulate all the required steps, and handle the toil of wiring up the Git repos, spinning up a CI/CD server, creating a PaaS to run the applications, instructing CI/CD to listen to the Git repos and push successful builds into the PaaS, and finally wiring applications to their required data services.

### Promise basics

Conceptually a Promise consists of three parts:

1. `workerClusterResources`: this is a collection of Kubernetes resources to be installed in the worker clusters.
1. `xaasCrd`: this is the CRD that is exposed to the users of the Promise.
1. `xaasRequestPipeline`: this is the pipeline that will create the resources requried to run an instance of the promised service on a worker cluster.

Let's take a closer look at these three pieces individually.

### The workerClusterResources

The `workerClusterResources` contains all the necessary Kubernetes resources you need to run the promised service. When a Promise is applied on the platform cluster, all objects under this section are applied to all worker clusters registered with the platform. For instance, if using the [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) in the Promise, this section will contain all the CRDs it needs plus the resources to run the operator itself.

### The xaasCrd

This is the user-facing API. It defines the configuration options exposed for users of the Promised service. Imagine the order form for a product. What do you need to know from your customer? Size? Location? Name?

This API can be as complex or as simple as you design it to be. You can read more about writing Custom Resource Definitions on the [Kubernetes docs](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#create-a-customresourcedefinition).

Users of the Platform will request a new instance of this Custom Object, as defined in the `xaasCrd`. The Custom Object definition is also referred to as _resource request_.

### The xaasRequestPipeline

The request pipeline is executed in response to a new resource request being received by the Kratix. It defines a set of jobs to run, with the aim to transform the request into the Kubernetes resources required to create an instance of the promised service.

The `xaasRequestPipeline` is an array of Docker images that execute in the defined order. It's expected that each container in the array will output complete, valid Kubernetes resources. This enables Promise writers to write specialised images and combine those as needed.

The contract with each pipeline container is simple and straightforward:
- The first container in the list receives the resource created by the user when they applied their request. This document, by definition, will be a valid Kubernetes resource as defined by the `xaasCrd`. The document will be available in `/input/object.yaml`.
- The container's command then executes, using the input object, and fulfilling any responsibilites necessary.
- The container writes any resources to be created to `/output/`.
- The resources in `/output` of the last container in the `xaasRequestPipeline` array will be scheduled and applied to the appropriate worker clusters.

In more advanced Promises, each of these 'stages' will take on responsibilities such as vulnerability scanning, licence checking, and secure certificate injection; the possibilities are endless. Look out for partnerships in this space to provide integrations for common services and tooling.


<br>
<hr>
<br>


# Writing your own Kratix Promise

- [Writing your own Kratix Promise](#writing-your-own-promise)
  - [What will I learn?](#what-will-i-learn)
  - [Writing a Promise](#writing-a-promise)
    - [Prerequisites:](#prerequisites)
    - [Promise template](#promise-template)
    - [X-as-a-Service Custom Resource Definition](#x-as-a-service-custom-resource-definition)
    - [X-as-a-Service Request Pipeline](#x-as-a-service-request-pipeline)
    - [Worker Cluster Resources](#worker-cluster-resources)
    - [Create and submit a resource request](#create-and-submit-a-resource-request)
  - [Summary](#summary)
  - [Where Next?](#where-next)

## What will I learn?
We will walk through the steps needed to create your own Promise, configure it for your needs, decorate it with your own opinions, and expose it as-a-Service ready for consumption by your platform users. If you are unsure of what a promise is and what problem it solves, check out [this page](https://github.com/syntasso/kratix/blob/main/docs/promises.md).

You will learn how to:
* Build a Promise for complex software, and expose it via a simple custom API which captures the data needed from users to configure the Promise for consumption "as-a-Service".
* Wrap and deploy the underlying Kubernetes CRDs, Operators, and resources required to run your Promise.
* Create a Promise pipeline to inject captured user-data into the underlying Kubernetes resources, and decorate the Promised software with custom behavior so the running Promise reflects your organisational and users' requirements.

## Writing a Promise


### Prerequisites:
* [Install Kratix across two KinD clusters](/installing-kratix/)


To begin writing a Promise you will need a basic directory structure to work in:

 ```
mkdir -p jenkins-promise/{resources,request-pipeline-image}
cd jenkins-promise
```


### Promise template

Create a basic `jenkins-promise-template.yaml` to work with:

```bash
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

You will fill the `spec` scalars as you progress through the tutorial.

### X-as-a-Service Custom Resource Definition
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

### X-as-a-Service Request Pipeline

Next, you will build the pipeline required to transform a Promise request into the Kubernetes resources required to create a running instance of the Jenkins service.

`cd request-pipeline-image`

Create the <code>jenkins-instance.yaml</code> by running the below command.
<details>
<summary><b>Note:</b> the code is folded for brevity</summary>

```bash
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


Kratix takes no opinion on the tooling used within a pipeline. Kratix will pass a set of resources to the pipeline, and expect back a set of resources. What happens within the pipeline, and what tooling is used, is a decision left entirely to the promise author. As the pipeline is simple (we're taking a name from the Promise custom resource input, and passing it to the Jenkins custom resource output) we're going to keep-it-simple and use a combination of `sed` and `yq` to do the work.

```bash
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
Then make it executable:

```bash
chmod +x execute-pipeline.sh
```

Next, you should create a simple `Dockerfile` that will copy the `jenkins-instance.yaml` doc into Kratix where it can be amended by the `execute-pipeline.sh` script and passed to the Worker cluster ready for execution.

```bash
cat > Dockerfile <<EOF
FROM "mikefarah/yq:4"
RUN [ "mkdir", "/tmp/transfer" ]

ADD jenkins-instance.yaml /tmp/transfer/jenkins-instance.yaml
ADD execute-pipeline.sh execute-pipeline.sh

CMD [ "sh", "-c", "./execute-pipeline.sh"]
ENTRYPOINT []
EOF
```

Next you will need to run
```bash
docker build . --tag <your-org-name/name-of-your-image>
```

You can test the container image by supplying an input resource and examining the output resource.

Let's create the test input and output directories locally.

```bash
mkdir {input,output}
```

The /input directory is where user resource request will be written when a user applies a resource to the Custom Resource Definition (`xaasCrd`) in the Promise (see above).

We need to add a sample user resource request to the /input directory:

```bash
cat > input/object.yaml <<EOF
apiVersion: promise.example.com/v1
kind: jenkins
metadata:
  name: my-jenkins-promise-request
spec:
  name: my-amazing-jenkins
EOF
```

Now you can run the container and examine the output.

```bash
docker run -v ${PWD}/input:/input -v ${PWD}/output:/output <your-org-name/name-of-your-image>
cat output/*
```

The contents of the /output directory will be scheduled and deployed by Kratix to a worker cluster. They need to be valid Kubernetes resources that can be applied to any cluster with the Promise's `workerClusterResources` installed (see beneath).

*If you already have the "Worker Cluster Resources" installed on a test cluster, you can optionally apply the resources in the /output directory and test to see if the Jenkins instance is created. This provides a fast feedback loop for developing a Promise. Continue to iterate on the container image, and running the image with input+output directories as above, until the files created in /output have the desired result when applied to a cluster with the worker cluster resources installed.*

Once you are satisified with the image, you can push it so it's ready for use in the pipeline.
```bash
docker push <your-org-name/name-of-your-image>
```

The final step of creating the `xaasRequestPipeline` is to add the  `spec.xaasRequestPipeline` scalar in `jenkins-promise-template.yaml` making sure you add the correct image details.

Let's move back to the "jenkins-promise" directory.

```bash
cd ..
```

And add the image to the array in `jenkins-promise-template.yaml`.

```yaml
xaasRequestPipeline:
    - <your-org-name/name-of-your-image>
```

In summary, you have:
- Created a container image containing:
    - A template file to be injected with per-instance details (jenkins-instance.yaml)
    - A shell script to retrieve the per-instance details from the user's request, and inject them into the template (execute-pipeline.sh)
    - A command set to the shell script
- Created a set of directories(input/output) and sample user request(input/object.yaml)
- Executed the pipeline image locally as a test
- Pushed the image to the registry
- Added the image to the Promise definition in the `xaasRequestPipeline` array

*Please note: at time of writing, only the first image in the `xaasRequestPipeline` array will be executed. Multiple image functionality will be available soon.*


### Worker Cluster Resources

 Jenkins.io has a [great Operator](https://jenkinsci.github.io/kubernetes-operator/docs/getting-started/latest/installing-the-operator/) that ships in two files.
1. [Jenkins CRDs](https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/config/crd/bases/jenkins.io_jenkins.yaml)
2. [The Operator](https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/deploy/all-in-one-v1alpha2.yaml) and other required resources such as Service Accounts, Role Bindings and Deployments.

We will need to download both.

```
wget https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/fbea1ed790e7a9deb2311e1f565ee93f07d89022/config/crd/bases/jenkins.io_jenkins.yaml -P resources
wget https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/8fee7f2806c363a5ceae569a725c17ef82ff2b58/deploy/all-in-one-v1alpha2.yaml -P resources
```

Next you need to inject Jenkins files into the `jenkins-promise-template.yaml`. To make this step simpler you have written a _very basic_ tool to grab all YAML documents from all YAML files located in `resources` and inject them into the `workerClusterResources` scalar.

```
go run path/to/kratix/hack/worker-resource-builder/main.go \
  -k8s-resources-directory ${PWD}/resources \
  -promise ${PWD}/jenkins-promise-template.yaml > jenkins-promise.yaml
```

This will create the finished `jenkins-promise.yaml` which can now be applied to the Kratix platform cluster:

```
kubectl apply --context kind-platform -f jenkins-promise.yaml
```

after a few seconds you can run `kubectl --context kind-platform get crds` and you should see something like:
```bash
NAME                          CREATED AT
jenkins.promise.example.com   2021-09-09T11:21:10Z
```

The complexities of what happens when installing a Promise are beyond this tutorial, but it's good to understand that a k8s Controller is now responding to Jenkins resource requests on the Platform cluster.

After a few minutes you can go to the Worker cluster and see the Jenkins operator running.

```
kubectl --context=kind-worker get pods -A
```

### Create and submit a resource request

Next, you change hats from Platform team member and become the customer of the Platform team. You should now be able to request instances of Jenkins on-demand.

```bash
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

After a few minutes the Jenkins operator will have received the request and asked the k8s worker to start an instance of Jenkins. You can target the Worker cluster (`kubectl config use-context kind-worker`) and run `kubectl get pods -A` to see the Jenkins instance with the defined name of `jenkins-my-amazing-jenkins`! (The Jenkins operator prepends the instance name with `jenkins-` hence `jenkins-my-amazing-jenkins`)

We can see the Jenkins UI in the browsers (all commands on worker cluster):
1. Get the username: `kubectl get secret jenkins-operator-credentials-my-amazing-jenkins -o 'jsonpath={.data.user}' | base64 -d`
2. Get the password: `kubectl get secret jenkins-operator-credentials-my-amazing-jenkins -o 'jsonpath={.data.password}' | base64 -d`
3. `kubectl port-forward jenkins-my-amazing-jenkins 8080:8080`
4. Navigate to http://localhost:8080

## Summary

We built a Jenkins-as-a-Service offering, by creating a Jenkins Promise, and adding the Promise to the Kratix platform.

We created the three elements of a Promise for Jenkins:
- `xaasCrd`
- `xaasRequestPipeline`
- `workerClusterResources`

and added them to the single Jenkins Promise yaml document. You then applied the Jenkins Promise to the platform cluster, which created the Jenkins-as-a-Service API, and configured the worker cluster such that it could create and manage Jenkins instances. Lastly, you assumed the role of a customer, and applied a yaml document to the Platform cluster, triggering the creation of a Jenkins instance on the Worker cluster.

### 🎉 &nbsp; Congratulations!
✅&nbsp;&nbsp; You have written a Kratix Promise. <br/>
👉🏾&nbsp;&nbsp; Let's [see how to tailor Kratix Promises based on organisational context](/enhancing-a-promise/README.md).

