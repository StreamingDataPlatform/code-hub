---
layout: post
category: Processing Data
tags: [stream processing, ingest, flink connector]
subtitle: To demo the usage of 'Streamcut'
img: ico-durability.png
license: Apache
support: Community
author: 
    name: Luis Liu, Youmin Han
    description: I'm focusing on the data stream solution development
    image: 
css: 
js: 
---
This sample application demonstrates the use of a Pravega abstraction, the `StreamCut`, in a Flink application to make 
it easier for developers to write analytics applications.
<!--more-->

## Purpose
The objective of this sample is to demonstrate that:  
**1.**   Flink applications may use `StreamCut` to bookmark points of interest within a Pravega `Stream`  
**2.**  `StreamCut` can be stored as persistent bookmarks for a Pravega stream  
**3.**  `StreamCut` may be used to perform bounded stream/batch processing on slices bookmarked by a Flink application.   

Note: Since this sample uses the local Flink environment, the instruction for Dell EMC Streaming Data Platform setup will be singly using **remote Pravega cluster** instead of deploying the application to SDP. However, it could be an academic exercise to enhance the example and run on Dell EMC SDP. 

## Design
The example consists of three applications: `DataProducer`, `StreamBookmarker` and `SliceProcessor`.  

```
$ cd flink-connector-examples/build/install/pravega-flink-examples
$ bin/dataProducer [--controller tcp://localhost:9090] [--num-events 10000]
$ bin/streamBookmarker [--controller tcp://localhost:9090] 
$ bin/sliceProcessor [--controller tcp://localhost:9090]
```

First, let us describe the role that these applications play in the sample:
- `DataProducer`: This application is intended to create data simulating various sensors publishing events. Each event
is a tuple `<sensorId, eventValue>`. In particular, the values of events follow a sinusoidal function to have intuitive
changing values within a controlled range for a demonstration.

- `StreamBookmarker`: This Flink application is intended to identify the slices of the `Stream` that we want to bookmark,
as they are of interest to execute further processing. That is, for a given `sensorId`, this job detects and stores the
start and end `StreamCut` (i.e., `Stream` "slice") for which `eventValues` are `< 0`. With the start and end `StreamCut`, 
this application creates a "slice" object that is persistently stored in a separate Pravega `Stream`.

- `SliceProcessor`: This application reads from the Pravega `Stream` where the start and end `StreamCut` are published by
`StreamBookmarker`. For each stream slice object written in the `Stream`, this application executes (locally) a batch job 
that reads the events within the specified slice and performs some computation (e.g., counting events for the sensor
for which values are `< 0`). This is an example of bounded processing in Flink on a Pravega `Stream`.

## Instructions
#### A. Running example on a local cluster with support from Pravega

##### 1. Setting up the Pravega and Flink environment
Before you start, you need to download the latest Pravega release on the [github releases page](https://github.com/pravega/pravega/releases). See [here](http://pravega.io/docs/latest/getting-started/) for the instructions to build and run Pravega in standalone mode.  

Besides, you also need to get the latest Flink binary from [Apache download page](https://flink.apache.org/downloads.html). Follow [this](https://ci.apache.org/projects/flink/flink-docs-stable/getting-started/tutorials/local_setup.html) tutorial to start a local Flink cluster. 

##### 2. Build `pravega-samples` Repository

`pravega-samples` repository provides code samples to connect analytics engines Flink with Pravega as a storage substrate for data streams. It has divided into sub-projects(`pravega-client-examples`, `flink-connector-examples` and `hadoop-connector-examples`), each one addressed to demonstrate a specific component. To build `pravega-samples` from source, follow the [instructions](https://github.com/pravega/pravega-samples#pravega-samples-build-instructions) to use the built-in gradle wrapper.  

##### 3. Start the Streamcuts Flink Example program

There are multiple ways to run the program in Flink environment including submitting from terminal or Flink UI. Follow the latest instruction from the [github README](https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples/doc/streamcuts) to learn more about the IDE setup and running process.

#### B. Running example on Dell EMC Streaming Data Platform

##### 1. Change the Streamcuts example code to satisfy the SDP running requirements
Since the original flink connector Streamcuts Flink example was designed to run on a standalone Pravega and Flink environment, the code used the `createScope` method from the StreamManager Interface in Pravega. However, due to the security reason, Dell EMC Streaming Data Platform does not allow to create a scope from the code. Please **comment out** `createScope` method from your code.   
There are three occurrences of `createScope` method in this Streamcuts Flink example which locate on following Java file:   
I. ```pravega-samples/flink-connector-examples/src/main/java/io/pravega/example/flink/streamcuts/process/DataProducer.java```  
II. ```pravega-samples/flink-connector-examples/src/main/java/io/pravega/example/flink/streamcuts/process/StreamBookmarker.java```
III. ```pravega-samples/flink-connector-examples/src/main/java/io/pravega/example/flink/streamcuts/process/SliceProcessor.java```  

##### 2. Follow the [this post]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html) to learn how to create Flink projects and run on Dell EMC Streaming Data Platform

The post for [Create Flink Project On Streaming Data Platform]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html) was designed for the Flink application running on Dell EMC Streaming Data Platform. Since this code sample only needs the Pravega cluster from the Dell EMC Streaming Data Platform, you can **skip Step C: Create Flink Clusters** through **Step F. Check Project Status** in the [post]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html).

In addition, since the code sample uses the default scope name, the **name/namespace** for this project **must** be ```examples```. You should have the same Analytics Projects Dashboard as following: 
![streamcut-project]({{site.baseurl}}/assets/heliumjk/images/post/streamcut/streamcut-project.png) 


##### 3. Connect the `DataProducer`, `StreamBookmarker`, and `SliceProcessor` application with the Pravega stream on Dell EMC Streaming Data Platform
After setting up the project namespace on SDP, configure Streaming Data Platform authentication by getting the ```keycloak.json``` file. Run the following command in your terminal: 
```
kubectl get secret examples-pravega -n examples -o jsonpath="{.data.keycloak\.json}" |base64 -d >  ${HOME}/keycloak.json

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
  "resource": "examples-pravega",
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

Before running the application, set the following environment variables. This can be done by setting the IntelliJ run configurations. (Go to run -> Edit Configurations -> Select DataProducer application -> Environment Variables -> click Browse icon. Fill the details mentioned below screen. Add all below program environment variables.) Make sure to change the `PRAVEGA_CONTROLLER` to the appropriate `EXTERNAL-IP` address. Notice that you may also need to set the same configurations for the `StreamBookmarker`, and `SliceProcessor` application.
```
pravega_client_auth_method=Bearer
pravega_client_auth_loadDynamic=true
KEYCLOAK_SERVICE_ACCOUNT_FILE=${HOME}/keycloak.json
PRAVEGA_CONTROLLER=tcp://<pravega controller external-ip>:9090
PRAVEGA_SCOPE=examples
```

Then you can save the configuration and run with the same order as discussed in the [github README](https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples/doc/streamcuts). Make sure you are using the `EXTERNAL-IP` address you get from above steps for the `--controller` parameter.

##### 4. Check the result after running Streamcuts Flink example
The result will be showing in your terminal or the console if you are using IntelliJ. You will find the same results as shown in the [github README](https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples/doc/streamcuts).

## Source
[https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples/doc/streamcuts](https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples/doc/streamcuts)