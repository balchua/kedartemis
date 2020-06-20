# KEDArtemis

Sample project to illustrate how to scale workloads based on [ActiveMQ Artemis](https://activemq.apache.org/components/artemis/) using [KEDA](keda.sh)

_ActiveMQ Artemis is the Next Generation Message Broker by ActiveMQ_

## Prerequisites

* Kubernetes cluster
* KEDA installed.  How to install KEDA, check out this [link](https://keda.sh/docs/1.4/deploy/)
* Skaffold - Optional, needed only when you want to build the consumer and producer programs.

## Code Organization

[`consumer`](consumer) - This is a simple SpringBoot project that listens to a queue named `test`.  It simply outputs the message 
into the console.

[`producer`](producer) - This is a process which pumps messages to the queue `test`.  This process runs to completion.
To run it in Kubernetes, you can use the `Job` resource, as shown in the example.  This job pumps `1000` messages.

[`k8s-manifests`](k8s-manifests) - This directory contains the necessary manifests to demonstrate the KEDA ActiveMQ Artemis Scaler.

Manifests:
* [`artemis-deployment`](k8s-manifests/artemis-deployment.yaml) - Deploys ActiveMQ Artemis into Kubernetes.  This uses the docker image `docker.io/vromero/activemq-artemis:2.6.2`.
You can checkout more at [vromero's github](https://github.com/vromero/activemq-artemis-docker) 

* [`consumer-deployment`](k8s-manifests/consumer-deployment.yaml) - Deploys the consumer process.  This starts with `0` replica.

* [`producer-job.yaml`](k8s-manifests/producer-job.yaml) - Runs to completion the producer job to pump messages into the queue. 

* [`scaledobject-secrets.yaml`](k8s-manifests/scaledobject-secrets.yaml) - This is the KEDA `ScaledObject` which controls how to scale the `consumer` workload. 
This also serves as a sample on how to populate the credentials needed to access Artemis' management endpoint.

* [`scaledobject-triggerauth.yaml`](k8s-manifests/scaledobject-triggerauth.yaml) - This is the KEDA `ScaledObject` which controls how to scale the `consumer` workload. 
This also serves as a sample on how to use KEDA's `TriggerAuthentication` to connect to Artemis' Management Endpoint to gather the queue metrics.

## Configuring ActiveMQ Artemis

To configure ActiveMQ Artemis, there is a `ConfigMap` defined in the file [`artemis-deployment.yaml`](k8s-manifests/artemis-deployment.yaml)
As an example to auto create the queue `test` with address `test`

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<configuration xmlns="urn:activemq" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:activemq /schema/artemis-configuration.xsd">

  <core xmlns="urn:activemq:core" xsi:schemaLocation="urn:activemq:core ">
     <name>artemis-activemq</name>
     <addresses>
       <address name="test">
       <anycast>
         <queue name="test"/>
       </anycast>
     </address>
    </addresses>
  </core>
</configuration>
```

If in any case you changed the broker address, you need to change the [`Consumer.java`](consumer/src/main/java/org/bal/consumer/Consumer.java) and change the `@JmsListener(destination = "test", concurrency = "1")`.
Same have to be done in the [`Producer.java`](producer/src/main/java/org/bal/producer/Producer.java) code.
 