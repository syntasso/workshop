# Enhancing the sample Postgres Promise

<!-- TOC -->

## What will I learn?

This document will walk through taking an "off the shelf" Promise and extending it to meet another set of requirements. While it is great to take advantage of community available Promises, you may require slightly different configuration options. For example, your Stream-Aligned Teams (SATs) have requested a database, and your team also has a requirement from the finance team to track costs for any new infrastructure.

In this example, we'll customise a sample Postgres Promise to satisfy this new requirement.

To make this change, we will need to become familiar with the existing Postgres Promise and then make a `costCentre` variable configurable which can be used to set a resource label for cost tracking. We will then assume the role of an application developer on a SAT, and request a new Postgres database while providing the new property.


## Platform team providing an enhanced Postgres Promise

<!-- Should there be a primer here about it being a platform engineer... -->


### Getting the sample Promise

We will use a sample Postgres Promise provided in the Kratix repository as the base of our custom Promise. Fro that, you will need to clone this repository and navigate to `samples/postgres` directory.

```bash
git clone https://github.com/syntasso/kratix.git
cd kratix/samples/postgres/
```

In this directory you will see a complete Promise as well as an example resource request. If you have not yet worked with Promises, you can learn more about the basic structure in Kratix's [Writing a Promise](https://github.com/syntasso/kratix/blob/main/docs/writing-a-promise.md) documentation.

<!-- This is a soft duplicated from `writing-a-promise` which will be hard to maintain -->
The Promise that will get installed into our Kubernetes cluster is in the `postgres-promise.yaml` file. A Promise is made of three parts:

* `xaasCrd`: this is the CRD that is exposed to the users of the Promise. It is the Platform team's contract with the SATs. Here is where we will introduce a `costCentre` property.
* `xaasRequestPipeline`: this is the pipeline that will create the resources required to run Postgres on a worker cluster. Here is where we'll use that `costCentre` property to set a label for cost tracking.
* `workerClusterResources`: this contains all of the Kubernetes resources required to run an instance of Postgres, such as CRDs, Operators and Deployments. This will not be required to change since the label is only applied to single instance of Postgres.

The `postgres-resource-request.yaml` file is an example of what SATs can use to request an postgres instance. As a application developer, we will need to update this request to include the newly defined `costCentre` property.


### Adding a new property to the xaasCrd

The contract with the SATs is defined by a number of properties on the `xaasCrd`. These properties are defined within a versioned schema and can have different types and validations. It's in this section that the platform team defines what are the required and optional configuration options exposed to the SATs.

In our case, we want to add a new required property called `costCentre` of type string and with a simple pattern requiring only certain character types. The complete property is as follows:

```yaml
costCentre:
  pattern: "^[a-zA-Z0-9_.-]*$"
  type: string
```
<!-- TODO: can we make this one required? -->

Add the property as a sibling property to the existing `preparedDatabases` property. To navigate to this, open the `postgres-promise.yaml` file, then find the `xaasCrd` section under `spec`. Within the xaasCrd find the `v1` version which has a set of properties within the `openAPIV3Schema` spec.

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


### Updating the xaasRequestPipeline to use the new property

While the `xaasCrd` allows us to accept a custom cost centre ID as input, we will need to use our custom request pipeline to add the correct cost centre ID as a label for our finance team to track costs.

The Postgres request pipeline has three parts, which you can find in the `request-pipeline-image` directory. Namely:

* `Dockerfile`: the image Kratix will execute when a new Postgres gets requested.
* `execute-pipeline.sh`: this is the script that will be executed (see the contents of the Dockerfile), where any logic or substitution we need should live.
* `minimal-postgres-manifest.yaml`: a basic Postgres manifest that can be understood by the Postgres operator. That's the pipeline basic template for a Postgres instance.

We will in turn take a look at all those files.


#### Introducing a new resource label to the Postgres manifest

When a new Postgres is requested, we need to generate a `postgresql` resource. The template for this resources is stored as `minimal-postgres-manifest.yaml`. In order to allow customisation of a label, we first need to set the label in this template by updating the metadata. Go ahead and add the following under `metadata`, taking care that it's correctly indented:

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

This manifest file will act as the "input" to the request pipeline script where we will inject the user configuration into the pre-defined fields. Let's proceed in updating the script to do just that.


#### Updating the pipeline script to set the new resource label from user input

As defined in the Dockerfile for the request pipeline, the `execute-pipeline.sh` script is where the pipeline logic lives. We will need to update this script to read the user input and set the right resource label. Looking at the current logic, we can see we are already parsing our resource request to identify key user variables, then using [yq](https://github.com/mikefarah/yq) to process the template file and replace certain fields with the user inputted values.

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


#### Testing it all together locally

Since a pipeline is just the manipulation of an input value to generate an output file, it can be easily validate locally by building and running the docker image with the correct volume mounts.

To set up this test, we will create two directories inside `request-pipeline-image`: `input` and `output`. Inside `input`, then we will add our expected input file (i.e., the resource request the SATs provide). From the `kratix/samples/postgres` directory, run the following:

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

Now we can then build this image with a custom tag:

_(to run this command, make sure you are within the `request-pipeline-image` directory)_
```bash
docker build . --tag kratix-workshop/postgres-request-pipeline:latest
docker run -v ${PWD}/input:/input -v ${PWD}/output:/output kratix-workshop/postgres-request-pipeline:latest
```

And finally, you we can validate the `output/output.yaml` file holds the manifest including all customised values. It should look like the example below. If your output is different, go back and check the files we touched. Repeat this process until you're satisfied with the output.

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


#### Preparing your Kubernetes environment

Before moving on, you will want to make sure to have an environment ready to run a Kratix. This includes having two clusters which can speak to each other, one named `platform` which includes both a Kratix and MinIO installation, and one called `worker` which includes a Flux CD installation. If you have not yet set this up, it is best for you to follow the [Kratix Quick Start: Part I documentation](https://github.com/syntasso/kratix/blob/main/docs/quick-start.md).

<details>
  <summary>Not sure if you are properly set up? Click here to see some commands to verify a local KinD deployment</summary>

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
kratix-worker-system   Active   23s
```
</details>


#### Accessing the new request pipeline container image from your cluster

Once you have made and validated all the pipeline image changes, you will need to make the newly created `kratix-workshop/postgres-pipeline-request` image accessible by your platform. This can be tricky since you will not be able to push an image to an organisation you do not own (`kratix-workshop`).

If you are running a local local KinD cluster we can take advantage of the fact that Kubernetes will always look for locally cached images first. By running the following command, you will load the image into local caches which will therefore stop any remote DockerHub calls:

```bash
kind load docker-image kratix-workshop/postgres-pipeline-request --name platform
```

Alternatively, if you need to pull from a remote source, you can re-tag the image with your own DockerHub repository and then push it for public use.

```bash
docker tag kratix-workshop/postgres-request-pipeline:latest <your-dockerhub-org>/postgres-request-pipeline:latest
docker push <your-dockerhub-org>/postgres-request-pipeline:latest
```


#### Setting the xaasRequestPipeline image to our new custom image

Now that the new image is built and available in our platform cluster, we can update the Promise to use the new image. For that, open the `postgres-promise.yaml` and update the `xaasRequestPipeline` to use the `kratix-workshop/postgres-pipeline-request:latest` instead of the `syntasso/postgres-pipeline-request`.

<details>
  <summary>Click here to see the resulting xaasRequestPipeline section which should be indented under `spec` in the Promise yaml</summary>

```yaml
xaasRequestPipeline:
  -  kratix-workshop/postgres-pipeline-request:latest
```

</details>


### Releasing the enhanced Promise to our platform

Once you have either loaded the image or updated your pipeline with the correct remote image, we are ready to release the Promise to our platform.



Once you verify that kratix is installed, you can simply apply our Promise yaml to the platform cluster:

```bash
kubectl --context kind-platform apply -f postgres-promise.yaml
```


## Stream-Aligned Team (SAT) requesting Postgres

Until now, we have been acting as a platform engineer designing, updating, and releasing a new Promise to enhance our platform. With this Promise now available, we are going to take a moment to switch hats and have a look at what one of our application developers on a SAT would do to take advantage of this new Promise.

### Updating the resource request

When the platform team install a promise into a cluster, they install a Custom Resource Definition which they describe under the `xaasCrd` section of the spec. This creates a new kind within the `example.promise.syntasso.io/v1` group of resources and includes all the details necessary for a customer application developer to request the resource. Like all Kubernetes resources, this request needs to include a unique name and namespace combination as well as any required fields in the spec.

As an application developer, we will need create our request in the platform cluster which will then use it to create a Postgres in the worker cluster for us to use. To make this request as an application developer, we can run the following command:

```bash
cat <<EOF | kubectl apply --context kind-platform -f -
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

### Validating the created Postgres


## Summary

<!--

What did we learn?

- Extended the xaasCrd to support the new costCentre property
- Created a new request pipeline to process the new property
- As an app dev, we updated the resource request to include the new costCentre property
- Validated the label got there

 -->

In this workshop, we explored the components that make up a Kratix Promise. We then customised an existing Postgres promise, tailoring it to our specific organisation needs.

We started by extending the Promise's `xaasCrd` which acts as the contract between the platform team and their users to accept a new property: `costCentre`. We defined its type and some basic validations using Schema object in the OpenAPI V3. The validation runs when a new resource request is received by the platform cluster. In a production environment, we will want to put more robust validations in place, like only accepting certain values.

A more complete list of the possible validation can be found [here](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.0.md#schemaObject).

Once the Promise's contract was updated to accept the `costCentre` property, we moved our attention to the Promise's `xaasRequestPipeline`. It's the pipeline that takes the resource request and outputs the set of resources that will be created in the worker cluster. In our example, we updated the pipeline script to set a new label on the resulting postgres based on the user's `costCentre` input.

While our example was very simple, it shows the power of Kratix Pipelines. You could, for example:

1. Send a request to an external API to validate the user sending the request has permission to bill that particular cost centre.
2. Verify any quotas that may have been setup, and fire an email to inform an interested party of this action

Furthermore, instead of editing the script being executed by the `postgres-request-pipeline` image, you could move logic from each step into its own dedicated image and just add these images to the `xaasRequestPipeline`. This would allow you to re-use the logic in all other Promises you publish in your platform.

We then switched hats and, as a member of a Stream-Aligned Team (SAT), sent out a resource request for a new Postgres instance. The request was very straightforward, we only had to add the new required property to the `spec` session of our resource requets.
<!-- Maybe we want to add a bit of future-looking here? -->
<!-- Add a few words on why this is good for app devs -->

Finally, we observed how everything works together by validating the a new Postgres instance was eventually created in our worker cluster, and that it had the right labels. We could now use whatever system we currently have in place to charge the cost centre for this new resource.


## What's next?

<!-- Maybe ack that we never touched the workerClusterResources section in this change? -->

Now that you have hands on experience working with Promises, you can start thinking about what Promises you would need on your organisation. A good next step is to learn more about how to design good Promises.

<!--

Have hands on experience with two working promises, now to learn how to scope / design a new one

-->