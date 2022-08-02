This is Part 5, the final hands-on part, of [a series](../README.md) illustrating how Kratix works. <br/>
👈🏾&nbsp;&nbsp; Previous: [Writing and installing a Kratix Promise](/writing-a-promise/) <br/>
👉🏾&nbsp;&nbsp; Next: [Final Thoughts](/final-thoughts/)

<hr> 

### In this tutorial, you will 
1. [learn more about the power in leveraging customised Kratix Promises]()
1. [enhance an off-the-shelf Postgres Promise]()

# How do I make Kratix work for my organisation?

As [we've seen](/using-multiple-promises/README.md), Kratix can support off-the-shelf Promises for services like Jenkins, Knative, and Postgres. 

The reality is that we have not worked with any organisation that is comfortable running production instances of these kinds of services without custom configuration&mdash;business rules like compliance, security, and billing need to live as configuration in these services.

As we'll see in the next hands-on section, Kratix and its Promises make these and other types of required configuration easy. 

### By using Kratix to build your platform, you can:
- save toil time in maintaining your own custom platform infrastructure.
- create a Platform as a Product that offers your "blessed" Golden Path services on demand.
- create your on-demand services from lower-level Kubernetes "operators".
- build and maintain your platform using familiar Kubernetes tools and native constructs.
- start small on a laptop and expand to multi-team, multi-cluster, multi-region, and multi-cloud with a consistent API everywhere.

### By offering a Kratix-built platform to your application teams, you allow developers to:
- discover available services that are already fit-for-purpose.
- consume services on demand using standard Kubernetes APIs.
- move focus away from infrastructure toward adding product value.

Now that you know more about how a customised Kratix platform can add value, let's enhance an off-the-shelf Kratix Promise following a set of business requirements.

<br>
<hr>
<br>

## Enhancing an off-the-shelf Postgres Promise

Table of Contents
=================

* [What will I learn?](#what-will-i-learn)
* [Platform team providing an enhanced Postgres Promise](#platform-team-providing-an-enhanced-postgres-promise)
  * [Getting the sample Promise](#getting-the-sample-promise)
  * [Adding a new property to the xaasCrd](#adding-a-new-property-to-the-xaascrd)
  * [Changing the cluster resources to include a new label](#changing-the-cluster-resources-to-include-a-new-label)
  * [Updating the xaasRequestPipeline to use the new property](#updating-the-xaasrequestpipeline-to-use-the-new-property)
      * [Introducing a new resource label to the Postgres manifest](#introducing-a-new-resource-label-to-the-postgres-manifest)
      * [Updating the pipeline script to set the new resource label from user input](#updating-the-pipeline-script-to-set-the-new-resource-label-from-user-input)
      * [Testing it all together locally](#testing-it-all-together-locally)
      * [Preparing your Kubernetes environment](#preparing-your-kubernetes-environment)
      * [Accessing the new request pipeline container image from your cluster](#accessing-the-new-request-pipeline-container-image-from-your-cluster)
      * [Setting the xaasRequestPipeline image to our new custom image](#setting-the-xaasrequestpipeline-image-to-our-new-custom-image)
  * [Releasing the enhanced Promise to our platform](#releasing-the-enhanced-promise-to-our-platform)
* [App developer requesting Postgres](#app-developer-requesting-postgres)
  * [Submitting the resource request](#submitting-the-resource-request)
  * [Validating the created Postgres](#validating-the-created-postgres)
* [Summary](#summary)
* [What's next?](#whats-next)
* [Feedback](#feedback)


### What will I learn?

This document will walk through taking an "off the shelf" Promise and extending it to meet another set of requirements. While it is great to take advantage of Promises available in the community, you may require different configuration options for your business.

Once we identify a Promise that meets our basic needs (i.e. deliver a database as a service), we will step through how to introduce our custom changes. We will then assume the role of an application developer, and request a Postgres database instance complete with our business specific customisations.


### Platform team providing an enhanced Postgres Promise

We will first assume the role of a Platform engineer who, after discussing with teams internal to their organisation, decides to provide Postgres-as-a-Service in their internal Kratix platform. To be compliant with other parts of the business, this engineer knows that any resources created need to be traceable back to a cost centre, so departments can be properly charged.

We can consider how resources are scanned for costs out of scope for this workshop. We can assume that there is already a process in place to scan for a `costCentre` label and manage the charge back. Therefore, to satisfy our finance team requirements, we will need to ensure that any database created through our Postgres Promise contains that label. In order to do that, in this workshop, we will:

1. Identify a good base Postgres Promise.
1. Define the contract with our users.
1. Configure the Operator support custom labels.
1. Understand the parts of the Promise, and update them accordingly.
1. Package and release our custom Postgres Promise on the platform.

The goal for this section is to get you familiar with the Promise components, and give you the confidence to build and test a Promise from scratch.


#### Getting the sample Promise

We will use a sample Postgres Promise provided in the Kratix repository as the base of our custom Promise. For that, you will need to clone the repository and navigate to `samples/postgres` directory.

```bash
git clone https://github.com/syntasso/kratix.git
cd kratix/samples/postgres/
```

In this directory you will see a complete Promise as well as an example resource request. If you have not yet worked with Promises, you can learn more about the basic structure in [Writing a Promise](../writing-a-promise/README.md), which is the previous step in this series.

The Promise that will get installed into our Kubernetes cluster is in the `postgres-promise.yaml` file. A Promise consists of three parts:

* `xaasCrd`: this is the CRD that is exposed to the users of the Promise. It is the Platform team's contract with the consumers of the platform. Here is where we will introduce a `costCentre` property.
* `xaasRequestPipeline`: this is the pipeline that will create the resources required to run Postgres on a worker cluster. Here is where we'll set the value for the `costCentre` label based on the user input.
* `workerClusterResources`: this contains all of the Kubernetes resources required to create an instance of Postgres, such as CRDs, Operators and Deployments. This is where we will tell the Postgres Operator to create instances with a `costCentre` label.

The `postgres-resource-request.yaml` file is an example of how to request a postgres instance from the platform. As an application developer, we will need to update this request to include the newly defined `costCentre` property.


#### Adding a new property to the xaasCrd

The contract with the app developers, i.e. the consumers of the platform, is defined by a number of properties in the `postgres-promise.yaml` file in the `xaasCrd` section. These properties are defined within a versioned schema and can have different types and validations. It's in this section that the platform team defines what are the required and optional configuration options exposed to the consumers.

In our case, we want to add a new required property called `costCentre` of type string and with a simple pattern requiring only certain character types. The complete property is as follows:

```yaml
costCentre:
  pattern: "^[a-zA-Z0-9_.-]*$"
  type: string
```

Add this cost centre property as a sibling to the existing `preparedDatabases` property. To navigate to the properties, open the `postgres-promise.yaml` file, then find the `xaasCrd` section and within the xaasCrd find the `v1` version which has a set of properties within the `openAPIV3Schema` spec.

<details>
  <summary>Click here to view a final version of the extended `xaasCrd` which should be indented so as to nest under the `spec` header</summary>

```yaml
xaasCrd:
  apiVersion: apiextensions.k8s.io/v1
  kind: CustomResourceDefinition
  metadata:
    name: postgreses.example.promise.syntasso.io
  spec:
    group: example.promise.syntasso.io
    names:
      kind: postgres
      plural: postgreses
      singular: postgres
    scope: Namespaced
    versions:
    - name: v1
      schema:
        openAPIV3Schema:
          properties:
            spec:
              properties:
                costCentre:
                  pattern: "^[a-zA-Z0-9_.-]*$"
                  type: string
                preparedDatabases:
                  additionalProperties:
                    properties:
                      defaultUsers:
                        type: boolean
                      extensions:
                        additionalProperties:
                          type: string
                        type: object
                      schemas:
                        additionalProperties:
                          properties:
                            defaultRoles:
                              type: boolean
                            defaultUsers:
                              type: boolean
                          type: object
                        type: object
                    type: object
                  type: object
              type: object
          type: object
      served: true
      storage: true
```
</details>
<br />

#### Changing the cluster resources to include a new label

When installing a Promise, there are two sides. On one side, the platform team is providing access to a capability via the `workerClusterResources`. On the other side, the platform users (the application developers), will request an instance of that capability via the `xaasPipeline` outputs.

The Postgres Promise we are using leverages [Zalando's Postgres Operator](https://github.com/zalando/postgres-operator) to provide Postgres-as-a-Service. This operator packages up the complexities of configuring Postgres into a manageable configuration format. One of its configuration options allows certain labels to be set on the resulting Pods. This is exactly the behaviour we require when delivering cost tracking across instances.

To enable the label feature, we need to set any desired labels in the [`inherited_labels`](https://github.com/zalando/postgres-operator/blob/master/docs/reference/operator_parameters.md#kubernetes-resources?:=inherited_labels) option in the Operators config. The value is a comma delimited list of labels that all instances created by the Operator are permitted to be set.

Note that this change is needed only because that's how the underlying Postgres Operator works. If the "off the shelf" Promise was using a different Postgres Operator, a different change may be required (or no change at all).

Update the `workerClusterResources` section of the `postgres-promise.yaml` file by adding `inherited_labels: costCentre` in alphabetical order to the `ConfigMap` named `postgres-operator`.

<details>
  <summary>Click here to see a complete ConfigMap resource after this change</summary>

```yaml
- apiVersion: v1
  data:
    api_port: "8080"
    aws_region: eu-central-1
    cluster_domain: cluster.local
    cluster_history_entries: "1000"
    cluster_labels: application:spilo
    cluster_name_label: cluster-name
    connection_pooler_image: registry.opensource.zalan.do/acid/pgbouncer:master-16
    db_hosted_zone: db.example.com
    debug_logging: "true"
    docker_image: registry.opensource.zalan.do/acid/spilo-13:2.0-p7
    enable_ebs_gp3_migration: "false"
    enable_master_load_balancer: "false"
    enable_pgversion_env_var: "true"
    enable_replica_load_balancer: "false"
    enable_spilo_wal_path_compat: "true"
    enable_team_member_deprecation: "false"
    enable_teams_api: "false"
    external_traffic_policy: Cluster
    inherited_labels: costCentre
    logical_backup_docker_image: registry.opensource.zalan.do/acid/logical-backup:v1.6.3
    logical_backup_job_prefix: logical-backup-
    logical_backup_provider: s3
    logical_backup_s3_bucket: my-bucket-url
    logical_backup_s3_sse: AES256
    logical_backup_schedule: 30 00 * * *
    major_version_upgrade_mode: manual
    master_dns_name_format: '{cluster}.{team}.{hostedzone}'
    pdb_name_format: postgres-{cluster}-pdb
    pod_deletion_wait_timeout: 10m
    pod_label_wait_timeout: 10m
    pod_management_policy: ordered_ready
    pod_role_label: spilo-role
    pod_service_account_name: postgres-pod
    pod_terminate_grace_period: 5m
    ready_wait_interval: 3s
    ready_wait_timeout: 30s
    repair_period: 5m
    replica_dns_name_format: '{cluster}-repl.{team}.{hostedzone}'
    replication_username: standby
    resource_check_interval: 3s
    resource_check_timeout: 10m
    resync_period: 30m
    ring_log_lines: "100"
    role_deletion_suffix: _deleted
    secret_name_template: '{username}.{cluster}.credentials'
    spilo_allow_privilege_escalation: "true"
    spilo_privileged: "false"
    storage_resize_mode: pvc
    super_username: postgres
    watched_namespace: '*'
    workers: "8"
  kind: ConfigMap
  metadata:
    name: postgres-operator
```
</details>
<br />

### Updating the xaasRequestPipeline to use the new property

While the `xaasCrd` allows us to accept a custom cost centre ID as input, we will need to use our custom request pipeline to add the correct cost centre ID as a label for our finance team to track costs.

The Postgres request pipeline has three parts, which you can find in the `request-pipeline-image` directory:

* `Dockerfile`: the image Kratix will execute when a new Postgres gets requested.
* `execute-pipeline.sh`: the script that will be executed (see the contents of the Dockerfile), where any logic or substitution lives.
* `minimal-postgres-manifest.yaml`: a basic Postgres manifest that can be understood by the Postgres operator. That's the pipeline basic template for a Postgres instance.

We will in turn take a look at all those files.


##### Introducing a new resource label to the Postgres manifest

When a new Postgres is requested, we need to generate a `postgresql` resource. The template for this resource is stored as `minimal-postgres-manifest.yaml`. In order to allow customisation of a label, we first need to set the label in this template by updating the metadata. Go ahead and add the following under `metadata`, taking care that it's correctly indented:

```yaml
labels:
  costCentre: TBD
```

<details>
<summary>Click here for the complete metadata section</summary>

```yaml
metadata:
  name: TBD
  namespace: default
  labels:
    costCentre: TBD
```
</details>
<br />

This manifest file will act as the "input" to the request pipeline script where we will inject the user configuration into the pre-defined fields. Let's proceed in updating the script to do just that.
<br />

##### Updating the pipeline script to set the new resource label from user input

As defined in the Dockerfile for the request pipeline, the `execute-pipeline.sh` script is where the pipeline logic lives. We need to update this script to read the user input and set the right resource label. Looking at the current logic, we can see we are already parsing our resource request to identify key user variables, then using [yq](https://github.com/mikefarah/yq) to process the template file and replace certain fields with the user inputted values.

Therefore, we can extend this script to also process the new cost centre ID. To do this, we will need to export another environment variable to store it (`export COST_CENTRE=$(yq eval '.spec.costCentre' /input/object.yaml)`) and a new line to process the replacement as a part of the pipeline (`.metadata.labels.costCentre = env(COST_CENTRE) |`).

<details>
  <summary>Click here to view an example of the final script</summary>

```bash
#!/bin/sh

set -x

# Store all input files in a known location
cp -r /tmp/transfer/* /input/

# Read current values from the provided resource request
export NAME=$(yq eval '.metadata.name' /input/object.yaml)
export NAMESPACE=$(yq eval '.metadata.namespace' /input/object.yaml)
export COST_CENTRE=$(yq eval '.spec.costCentre' /input/object.yaml)
export PREPARED_DBS=$(yq eval '.spec.preparedDatabases' /input/object.yaml)

# Replace defaults with user provided values
cat /input/minimal-postgres-manifest.yaml |  \
  yq eval '.metadata.name = env(NAME) |
          .metadata.namespace = env(NAMESPACE) |
          .metadata.labels.costCentre = env(COST_CENTRE) |
          .spec.preparedDatabases = env(PREPARED_DBS)' - \
  > /output/output.yaml
```
</details>
<br />

#### Testing it all together locally

Since a pipeline is just the manipulation of an input value to generate an output file, it can be easily validated locally by building and running the docker image with the correct volume mounts.

To set up this test, we will create two directories inside `request-pipeline-image`: `input` and `output`. Inside `input`, then we will add our expected input file (i.e., the resource request the app developer provides). From the `kratix/samples/postgres` directory, run the following:

```bash
cd request-pipeline-image
mkdir -p {input,output}
cat > input/object.yaml <<EOF
---
apiVersion: example.promise.syntasso.io/v1
kind: postgres
metadata:
  name: acid-minimal-cluster
  namespace: default
spec:
  costCentre: "rnd-10002"
  preparedDatabases:
    mydb: {}
EOF
```

Now we can build this image with a custom tag:

_(to run this command, make sure you are within the `request-pipeline-image` directory)_
```bash
docker build . --tag kratix-workshop/postgres-request-pipeline:dev
docker run -v ${PWD}/input:/input -v ${PWD}/output:/output kratix-workshop/postgres-request-pipeline:dev
```

And finally, we can validate the `output/output.yaml` file holds the manifest including all customised values. It should look like the example below. If your output is different, go back and check the files we touched. Repeat this process until you're satisfied with the output.

<details>
    <summary>Expected output.yaml</summary>

```yaml
apiVersion: "acid.zalan.do/v1"
kind: postgresql
metadata:
  name: acid-minimal-cluster
  namespace: default
  labels:
    costCentre: rnd-10002
spec:
  teamId: "acid"
  volume:
    size: 1Gi
  numberOfInstances: 2
  users:
    zalando: # database owner
      - superuser
      - createdb
    foo_user: [] # role for application foo
  databases:
    foo: zalando # dbname: owner
  preparedDatabases:
    mydb: {}
  postgresql:
    version: "13"
```
</details>
<br />

#### Preparing your Kubernetes environment

Before moving on, you will want to make sure to have an environment ready to run Kratix. This includes having two clusters which can speak to each other, one named `platform` which includes both a Kratix and MinIO installation, and one called `worker` which includes a Flux CD installation. If you have not yet set this up, follow the [Quick Start: Install Kratix](../installing-kratix/README.md), which is the first step in this series.

<details>
  <summary>Not sure if you are properly set up? Click here to see commands to verify a local KinD deployment</summary>

To verify your have at least the two necessary clusters:
```console
$ kind get clusters
platform
worker
```

To verify Kratix and MinIO are installed and healthy:
```console
$ kubectl --context kind-platform get pods -n kratix-platform-system
NAME                                                  READY   STATUS       RESTARTS   AGE
kratix-platform-controller-manager-769855f9bb-8srtj   2/2     Running      0          1h
minio-6f75d9fbcf-5cn7w                                1/1     Running      0          1h
```

To verify we can deploy resources to the worker, we can check if our "canary" resource has been deployed:
```console
$  kubectl --context kind-worker get namespaces kratix-worker-system
NAME                   STATUS   AGE
kratix-worker-system   Active   1h
```
</details>
<br />

#### Accessing the new request pipeline container image from your cluster

Once you have made and validated all the pipeline image changes, you will need to make the newly created `kratix-workshop/postgres-request-pipeline:dev` image accessible by your platform. This can be tricky since you will not be able to push an image to an organisation you do not own (`kratix-workshop`).

If you are running a local local KinD cluster we can take advantage of the fact that Kubernetes will always look for locally cached images first. By running the following command, you will load the image into local caches which will therefore stop any remote DockerHub calls:

```bash
kind load docker-image kratix-workshop/postgres-request-pipeline:dev --name platform
```

Alternatively, if you need to pull from a remote source, you can re-tag the image with your own DockerHub repository and then push it for public use.

```bash
docker tag kratix-workshop/postgres-request-pipeline:dev <your-dockerhub-org>/postgres-request-pipeline:dev
docker push <your-dockerhub-org>/postgres-request-pipeline:dev
```


#### Setting the xaasRequestPipeline image to our new custom image

Now that the new image is built and available in our platform cluster, we can update the Promise to use the new image. For that, open the `postgres-promise.yaml` and update the `xaasRequestPipeline` to use the `kratix-workshop/postgres-request-pipeline:dev` instead of the `syntasso/postgres-request-pipeline`.

<details>
  <summary>Click here to see the resulting xaasRequestPipeline section which should be indented under `spec` in the Promise yaml</summary>

```yaml
xaasRequestPipeline:
  -  kratix-workshop/postgres-request-pipeline:dev
```

</details>
<br />

### Releasing the enhanced Promise to our platform

Once you have either loaded the image or updated your pipeline with the correct remote image, we are ready to release the Promise to our platform:

_(This command needs to be run from inside the `postgres` directory)_

```bash
kubectl --context kind-platform apply -f postgres-promise.yaml
```

This promise has been successfully installed once the promise is available:

```console
$ kubectl --context kind-platform -n default get promises
NAME                  AGE
ha-postgres-promise   1m
```

And the `workerClusterResources` have been installed. These resources are what must be present in the clusters for an instance of our Promise to be successfully provisioned. They are installed as soon as the Promise is added to the platform. For Postgres, we can see in the Promise file that there are a number of RBAC resources, as well as a deployment that installs the Postgres Operator in the worker cluster. That means that, when the Promise is successfully applied, we will see the `postgres-operator` deployment in the worker cluster. That's also an indication that the operator is ready to provision a new instance.

```console
$ kubectl --context kind-worker --namespace default get pods
NAME                                 READY   STATUS    RESTARTS   AGE
postgres-operator-6c6dbd4459-hcsg2   1/1     Running   0          1m
```

And that's it! You have successfully released a new platform capability! Let's move on to how teams can use this to request a new Postgres instance from the platform.

## App developer requesting Postgres

Until now, we have been acting as a platform engineer designing, updating, and releasing a new Promise to enhance our platform. With this Promise now available, we are going to take a moment to switch hats and have a look at what one of our application developers would do to take advantage of this new Promise.

### Submitting the resource request

As an application developer, we will need to create a resource request in the platform cluster. Like all Kubernetes resources, this request needs to include:

1. An API that the resource can be found under. This is `example.promise.syntasso.io/v1` in our Postgres promise (see `spec.xaasCrd.spec.group` in the Promise manifest).
1. A kind which points to a specific promise. In this case it will be `postgres` (see `spec.xaasCrd.spec.name` in the Promise manifest).
1. A unique name and namespace combination.
1. Any required fields in defined the in our Promise `spec`. These can be found in the `xaasCrd` section of the Promise (more specifically under the `openAPIV3Schema` spec).

You can start with the provided sample `postgres-resource-request.yaml` and add the additional `costCentre` field as a sibling to the `preparedDatabases` field with any valid input. For example, `costCentre: "rnd-10002"`.

<details>
<summary>Click here for the full Postgres resource request</summary>

```yaml
apiVersion: example.promise.syntasso.io/v1
kind: postgres
metadata:
  name: acid-minimal-cluster
  namespace: default
spec:
  costCentre: "rnd-10002"
  preparedDatabases:
    mydb: {}
```
</details>
<br />

Then apply this file to the platform cluster with the following command:

```bash
kubectl --context kind-platform apply --file postgres-resource-request.yaml
```

### Validating the created Postgres

As a platform engineer, we use our pipeline to support two different requirements when fulfilling the Postgres Promise.

Once the resource request is applied on the platform cluster, you should eventually see a new pod executing the pipeline script we just created. Listing the pods should output something similar to:

```console
$ kubectl --context kind-platform get pods
NAME                                                     READY   STATUS      RESTARTS   AGE
request-pipeline-ha-postgres-promise-default-<SHA>       0/1     Completed   0          1h
```

You can then view the pipeline logs by running:

_(make sure to update the pod SHA accordingly to the output of the `get pods` above)_

```bash
kubectl logs -c xaas-request-pipeline-stage-1 pods/request-pipeline-ha-postgres-promise-default-<SHA>
```

Once the pipeline is completed, you will eventually see on the worker cluster a Postgres service as a two pod cluster in `Running` state with the name we defined in our request:

```console
$ kubectl --context kind-worker get pods
NAME                                 READY   STATUS    RESTARTS   AGE
acid-minimal-cluster-0               1/1     Running   0          1h
acid-minimal-cluster-1               1/1     Running   0          1h
...
```

In addition, the pods will provide cost tracking for the finance team through a new label. This can be confirmed by only selecting pods that contain the provided cost centre value:

```console
$  kubectl --context kind-worker get pods --selector costCentre=rnd-10002
NAME                     READY   STATUS    RESTARTS   AGE
acid-minimal-cluster-0   1/1     Running   0          1h
acid-minimal-cluster-1   1/1     Running   0          1h
```

### Summary

In this workshop, we explored the components that make up a Kratix Promise. We then customised an "off the shelf" Postgres promise, tailoring it to our specific organisation needs before providing it on our platform.

We started by extending the Promise's `xaasCrd`, which acts as the contract between the platform team and their users, to accept a new property: `costCentre`. We defined its type and added some basic validations using the Schema object in the OpenAPI V3 specification.

We set a property on the Postgres Operator to add this custom `costCentre` label onto all resources it creates. This Operator is how the platform team standardises the creation of Postgres instances for each application development team. Part of our platform design is deciding which of these properties are exposed to our users, and which are set as standard for all Postgres instances created.

Once the Promise's contract was updated to accept the `costCentre` property, and the Postgres Operator was updated to use a custom label, we moved our attention to the Promise's `xaasRequestPipeline`. In our example, we updated the pipeline script to set a new label on the resulting postgres based on the user's `costCentre` input.

We then switched hats and, as a member of an application development team, sent out a resource request for a new Postgres instance. The request was straightforward, we only had to add the new required property to the `spec` session of our resource requests.

Finally, we observed how everything works together by validating the a new Postgres instance was eventually created in our worker cluster, and that it had the right labels. We could now use whatever system we currently have in place to charge the cost centre for this new resource.


### 🎉 &nbsp; Congratulations! 
✅&nbsp;&nbsp; You have enhanced a Kratix Promise to suit your organisation's needs. This concludes our introduction to Kratix. <br/>
👉🏾&nbsp;&nbsp; Let's [see where to go from here](/final-thoughts/README.md).

