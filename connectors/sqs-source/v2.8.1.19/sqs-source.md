---
description: 
author: freeznet,Anonymitaet,nlu90,danpi
contributors: freeznet,Anonymitaet,nlu90,danpi
language: Java,Shell,Python,Dockerfile
document:
source: "https://github.com/streamnative/pulsar-io-sqs"
license: Apache License 2.0
license_link: "https://github.com/streamnative/pulsar-io-sqs/blob/master/LICENSE"
tags: 
alias: AWS SQS Source Connector
features: [""]
icon: "https://avatars.githubusercontent.com/u/44651383?v=4"
download: "https://api.github.com/repos/streamnative/pulsar-io-sqs/tarball/refs/tags/v2.8.1.19"
support: streamnative
support_link: https://github.com/streamnative/pulsar-io-sqs
support_img: "https://avatars.githubusercontent.com/u/44651383?v=4"
owner_name: "streamnative"
owner_img: "https://avatars.githubusercontent.com/u/44651383?v=4"
dockerfile: "https://hub.docker.com/r/streamnative/pulsar-io-sqs"
id: "sqs-source"
---


The [AWS Simple Queue Service (SQS)](https://aws.amazon.com/sqs/?nc1=h_ls) source connector feeds data from Amazon AWS SQS and writes data to Pulsar topics.

![](https://raw.githubusercontent.com/streamnative/pulsar-io-sqs/v2.8.1.19/docs/sqs-source.png)

# How to get 

You can get the SQS source connector using one of the following methods.

## Use it with Function Worker

- Download the NAR package from [here](https://github.com/streamnative/pulsar-io-sqs/releases/download/v2.8.0-rc-202106091215/pulsar-io-sqs-2.8.0-rc-202106091215.nar).

- Build it from the source code.

  1. Clone the source code to your machine.

     ```bash
     git clone https://github.com/streamnative/pulsar-io-sqs
     ```

  2. Assume that `PULSAR_IO_SQS_HOME` is the home directory for the `pulsar-io-sqs` repo. Build the connector in the `${PULSAR_IO_SQS_HOME}` directory.

     ```bash
     mvn clean install -DskipTests
     ```

     After the connector is successfully built, a `NAR` package is generated under the `target` directory. 

     ```bash
     ls target
     pulsar-io-sqs-2.8.0-rc-202106091215.nar
     ```

## Use it with Function Mesh

Pull the SQS connector Docker image from [here](https://hub.docker.com/r/streamnative/pulsar-io-sqs).

# How to configure 

Before using the SQS source connector, you need to configure it. Below are the properties and their descriptions.

| Name | Type|Required | Default | Description
|------|----------|----------|---------|-------------|
| `awsEndpoint` |String| false | "" (empty string) | AWS SQS end-point URL. It can be found at [here](https://docs.aws.amazon.com/general/latest/gr/sqs-service.html#sqs_region). |
| `awsRegion` | String| true | " " (empty string) | Supported AWS region. For example, us-west-1, us-west-2. |
| `awsCredentialPluginName` | String|false | " " (empty string) | Fully-qualified class name of implementation of `AwsCredentialProviderPlugin`. |
| `awsCredentialPluginParam` | String|true | " " (empty string) | JSON parameter to initialize `AwsCredentialsProviderPlugin`. |
| `queueName` | String|true | " " (empty string) | Name of the SQS queue that messages should be read from or written to. |

## Configure it with Function Worker

You can create a configuration file (JSON or YAML) to set the properties as below.

**Example**

* JSON 

   ```json
    {
        "tenant": "public",
        "namespace": "default",
        "name": "sqs-source",
        "topicName": "test-queue-pulsar",
        "archive": "connectors/pulsar-io-sqs-2.8.0-rc-202106091215.nar",
        "parallelism": 1,
        "configs":
        {
            "awsEndpoint": "https://sqs.us-west-2.amazonaws.com",
            "awsRegion": "us-west-2",
            "queueName": "test-queue",
            "awsCredentialPluginName": "",
            "awsCredentialPluginParam": '{"accessKey":"myKey","secretKey":"my-Secret"}'
        }
    }
    ```

* YAML

   ```yaml
   tenant: "public"
   namespace: "default"
   name: "sqs-source"
   topicName: "test-queue-pulsar"
   archive: "connectors/pulsar-io-sqs-2.8.0-rc-202106091215.nar"
   parallelism: 1

   configs:
      awsEndpoint: "https://sqs.us-west-2.amazonaws.com"
      awsRegion: "us-west-2"
      queueName: "test-queue"
      awsCredentialPluginName: ""
      awsCredentialPluginParam: '{"accessKey":"myKey","secretKey":"my-Secret"}'
    ```

## Configure it with Function Mesh

You can submit a [CustomResourceDefinitions (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) to create an SQS source connector. Using CRD makes Function Mesh naturally integrate with the Kubernetes ecosystem. For more information about Pulsar source CRD configurations, see [here](https://functionmesh.io/docs/connectors/io-crd-config/source-crd-config).

You can define a CRD file (YAML) to set the properties as below.

```yaml
apiVersion: compute.functionmesh.io/v1alpha1
kind: Source
metadata:
  name: sqs-source-sample
spec:
  image: streamnative/pulsar-io-sqs:2.8.0-rc-202106091215
  className: org.apache.pulsar.ecosystem.io.sqs.SQSSource
  replicas: 1
  maxReplicas: 1
  output:
    topic: persistent://public/default/destination
    typeClassName: “[B”
  sourceConfig:
    awsEndpoint: "https://sqs.us-west-2.amazonaws.com"
    awsRegion: "us-west-2"
    queueName: "test-queue"
    awsCredentialPluginName: ""
    awsCredentialPluginParam: '{"accessKey":"myKey","secretKey":"my-Secret"}'
  pulsar:
    pulsarConfig: "test-pulsar-source-config"
  resources:
    limits:
    cpu: "0.2"
    memory: 1.1G
    requests:
    cpu: "0.1"
    memory: 1G
  java:
    jar: connectors/pulsar-io-sqs-2.8.0-rc-202106091215.nar
  clusterName: test-pulsar
```

# How to use

You can use the SQS source connector with Function Worker or Function Mesh.

## Use it with Function Worker

You can use the SQS source connector as a non built-in connector or a built-in connector.

### Use it as non built-in connector

If you already have a Pulsar cluster, you can use the SQS source connector as a non built-in connector directly.

This example shows how to create an SQS source connector on a Pulsar cluster using the [`pulsar-admin sources create`](http://pulsar.apache.org/tools/pulsar-admin/2.8.0-SNAPSHOT/#-em-create-em--14) command.

```
PULSAR_HOME/bin/pulsar-admin sources create \
--archive pulsar-io-sqs-2.8.0-rc-202106091215.nar \
--source-config-file sqs-source-config.yaml \
--classname org.apache.pulsar.ecosystem.io.sqs.SQSSource \
--name sqs-source
```

### Use it as built-in connector

You can make the SQS source connector as a built-in connector and use it on a standalone cluster or on-premises cluster.

#### Standalone cluster

This example describes how to use the SQS source connector to feed data from SQS and write data to Pulsar topics in standalone mode.

1. Prepare SQS service. 
 
    For more information, see [Getting Started with Amazon SQS](https://aws.amazon.com/sqs/getting-started/).
 
2. Copy the NAR package to the Pulsar connectors directory.
 
    ```
    cp pulsar-io-sqs-2.8.0-rc-202106091215.nar PULSAR_HOME/connectors/pulsar-io-sqs-2.8.0-rc-202106091215.nar
    ```

3. Start Pulsar in standalone mode.

    ```
    PULSAR_HOME/bin/pulsar standalone
    ```

4. Run the SQS source connector locally.

    ```
    PULSAR_HOME/bin/pulsar-admin sources localrun \
    --source-type sqs  \
    --source-config-file sqs-source-config.yaml
    ```

5. Consume the message from the Pulsar topic.
    
    ```
    PULSAR_HOME/bin/pulsar-client consume -s "sub-products" public/default/test-queue-pulsar -n 0
    ```

6. Send a message to the SQS queue using the [AWS SQS CLI tool](https://aws.amazon.com/cli/).

    ```
    aws sqs send-message --queue-url ${QUEUE_URL} --message-body "Hello From SQS"
    ```

    Now you can see the message "Hello From SQS" from the Pulsar consumer.

#### On-premises cluster

This example explains how to create an SQS source connector in an on-premises cluster.

1. Copy the NAR package of the SQS connector to the Pulsar connectors directory.

    ```
    cp pulsar-io-sqs-2.8.0-rc-202106091215.nar $PULSAR_HOME/connectors/pulsar-io-sqs-2.8.0-rc-202106091215.nar
    ```

2. Reload all [built-in connectors](https://pulsar.apache.org/docs/en/next/io-connectors/).

    ```
    PULSAR_HOME/bin/pulsar-admin sources reload
    ```

3. Check whether the SQS source connector is available on the list or not.

    ```
    PULSAR_HOME/bin/pulsar-admin sources available-sources
    ```

4. Create an SQS source connector on a Pulsar cluster using the [`pulsar-admin sources create`](http://pulsar.apache.org/tools/pulsar-admin/2.8.0-SNAPSHOT/#-em-create-em--14) command.

    ```
    PULSAR_HOME/bin/pulsar-admin sources create \
    --source-type sqs \
    --source-config-file sqs-source-config.yaml \
    --name sqs-source
    ```

## Use it with Function Mesh

This example demonstrates how to create an SQS source connector through Function Mesh.

### Prerequisites

- Create and connect to a [Kubernetes cluster](https://kubernetes.io/).

- Create a [Pulsar cluster](https://pulsar.apache.org/docs/en/kubernetes-helm/) in the Kubernetes cluster.

- [Install the Function Mesh Operator and CRD](https://functionmesh.io/docs/install-function-mesh/) into the Kubernetes cluster.

- Prepare SQS service. 

  For more information, see [Getting Started with Amazon SQS](https://aws.amazon.com/sqs/getting-started/).

### Step

1. Define the SQS source connector with a YAML file and save it as `source-sample.yaml`.

    This example shows how to publish the SQS source connector to Function Mesh with a Docker image.

    ```yaml
    apiVersion: compute.functionmesh.io/v1alpha1
    kind: Source
    metadata:
    name: sqs-source-sample
    spec:
    image: streamnative/pulsar-io-sqs:2.8.0-rc-202106091215
    className: org.apache.pulsar.ecosystem.io.sqs.SQSSource
    replicas: 1
    maxReplicas: 1
    output:
        topic: persistent://public/default/destination
        typeClassName: “[B”
    sourceConfig:
        awsEndpoint: "https://sqs.us-west-2.amazonaws.com"
        awsRegion: "us-west-2"
        queueName: "test-queue"
        awsCredentialPluginName: ""
        awsCredentialPluginParam: '{"accessKey":"myKey","secretKey":"my-Secret"}'
    pulsar:
        pulsarConfig: "test-pulsar-source-config"
    resources:
        limits:
        cpu: "0.2"
        memory: 1.1G
        requests:
        cpu: "0.1"
        memory: 1G
    java:
        jar: connectors/pulsar-io-sqs-2.8.0-rc-202106091215.nar
    clusterName: test-pulsar
    ```

2. Apply the YAML file to create the SQS source connector.

    **Input**

    ```
    kubectl apply -f  <path-to-source-sample.yaml>
    ```

    **Output**

    ```
    source.compute.functionmesh.io/sqs-source-sample created
    ```

3. Check whether the SQS source connector is created successfully.

    **Input**

    ```
    kubectl get all
    ```

    **Output**

    ```
    NAME                                READY   STATUS      RESTARTS   AGE
    pod/sqs-source-sample-0               1/1     Running     0          77s
    ```

    After that, you can produce and consume messages using the SQS source connector between Pulsar and SQS.


