---
layout: post
category: "Getting Started"
tags: [flink, java, SDP, Pravega]
subtitle: "Pravega Stream Ingest on Dell EMC Streaming Data Platform"
img: pravega-stream.png
license: Apache
support: Community
author: 
    name: Youmin Han
    description: Nautilus App Developer
    image: batman.png
css: 
js: 
---
This is a general guide for writing and ingesting the Pravega stream on Dell EMC Streaming Data Platform.
<!--more-->

## Purpose

The Dell EMC Streaming Data Platform architecture contains [Pravega](https://www.pravega.io/), an open-source streaming storage system that implements streams and acts as first-class primitive for storing or serving continuous and unbounded data. Pravega [streams](https://pravega.io/docs/latest/pravega-concepts/#streams) are based on an append-only log-data structure that rapidly ingests data into the durable storage

## Design
The simplest kind of Pravega application uses a Reader to read from a Stream or a Writer that writes to a Stream. For the Pravega Stream Ingest we will mainly focus on how to write the data to Pravega Stream on Dell EMC Streaming Data Platform. A simple sample application of both can be found in the [workshop-samples repository](https://github.com/pravega/workshop-samples/tree/master/stream-ingest/src/main/java/com/dellemc/oe/ingest). These samples give a sense on how a Java application could use the Pravega's Java Client Library to access Pravega functionality.

The example consists of four applications: `EventWithTimestampWriter`, `EventWriter`, `ImageWritersor`, and `JSONWriter`.  

All applications demonstrate the usage of EventStreamWriter to write an Event to Pravega and have the idential ways to write the data to Pravega stream on Dell EMC SDP. The key part is in the `run()` method which creates a Stream and outputs the given Event to that Stream.

## Instructions
##### 1. Follow the [this post]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html) to learn how to create Flink projects and run on Dell EMC Streaming Data Platform

The post for [Create Flink Project On Streaming Data Platform]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html) was designed for the Flink application running on Dell EMC Streaming Data Platform. In order to write the data to Pravega stream on Dell EMC Streaming Data Platform, you need to complete **Step A: Remove `createScope` method from the code** till **Step C. Create Flink Clusters** in the [post]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html).

Notice that the **project name** you created will be **same** as the **Pravega scope/namespace** and will be used in the later steps. You should have an Analytics Project showing on your Dashboard as following (project name may vary): 
![streamcut-project]({{site.baseurl}}/assets/heliumjk/images/post/streamcut/streamcut-project.png) 

##### 2. Connect the applications with the Pravega srteam on Dell EMC Streaming Data Platform
After setting up the project namespace on SDP, configure Streaming Data Platform authentication by getting the ```keycloak.json``` file. Replace following `<pravega scope>` with the **project name** you set in the previous step and run the command in your terminal: 
```
kubectl get secret <pravega scope>-pravega -n <pravega scope> -o jsonpath="{.data.keycloak\.json}" |base64 -d >  ${HOME}/keycloak.json

chmod go-rw ${HOME}/keycloak.json
```
The output should look like the following:
```
{
  "realm": "nautilus",
  "auth-server-url": "https://keycloak.p-test.nautilus-lab-wachusett.com/auth",
  "ssl-required": "external",
  "bearer-only": false,
  "public-client": false,
  "resource": "<pravega scope>-pravega",
  "confidential-port": 0,
  "credentials": {
    "secret": "c72c45f8-76b0-4ca2-99cf-1f1a03704c4f"
  }
}
```
In order to access the Pravega cluster from Dell EMC Streaming Data Platform, you need to connect with Pravega controller. Use `kubectl` to get the `EXTERNAL-IP` of the nautilus-pravega-controller service.
```
kubectl get service nautilus-pravega-controller -n nautilus-pravega

NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                          AGE
nautilus-pravega-controller   LoadBalancer   10.100.200.243   11.111.11.111   10080:32217/TCP,9090:30808/TCP   6d5h
```

Before running the application, set the following environment variables. This can be done by setting the IntelliJ run configurations. (Go to run -> Edit Configurations -> Select JSONWriter application -> Environment Variables -> click Browse icon. Fill the details mentioned below screen. Add all below program environment variables.) Make sure to change the `PRAVEGA_CONTROLLER` to the appropriate `EXTERNAL-IP` address. Notice that the assigned value for `PRAVEGA_SCOPE` and `PRAVEGA_STREAM` may need to be changed based on your settings when created the Flink project and application on SDP. You also need to set the same configurations for the `EventWithTimestampWriter`, `EventWriter`, and `ImageWritersor` application.  
```
pravega_client_auth_method=Bearer
pravega_client_auth_loadDynamic=true
KEYCLOAK_SERVICE_ACCOUNT_FILE=${HOME}/keycloak.json
PRAVEGA_CONTROLLER=tcp://<pravega controller external-ip>:9090
PRAVEGA_SCOPE=<pravega scope>
PRAVEGA_STREAM=<target stream name>
```

In addition, if your have a reader running on Dell EMC SDP at the same time, please make sure to pass the same Pravega scope and stream parameters. More details are discussed in [original post](https://github.com/pravega/workshop-samples#running-jsonwriter-from-intelij).     

Then you can save the configuration and hit Run.


##### 3. Check the result after running application
The result will print in your terminal or the console if you are using IntelliJ. You will find the Bytes Written are showing up on the Pravega stream dashboard as following. 
![stream-dashboard]({{site.baseurl}}/assets/heliumjk/images/post/pravega-ingest/stream-dashboard.png) 



## Source
[Pravega Workshop Samples](https://github.com/pravega/workshop-samples)


