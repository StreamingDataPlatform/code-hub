---
layout: post
category: "Getting Started"
tags: [flink, java, sdp, pravega]
subtitle: "Pravega Stream Ingest on Dell EMC Streaming Data Platform"
technologies: [SDP, Pravega]
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
This is a general guide for writing and ingesting into a Pravega stream on Dell EMC Streaming Data Platform.
<!--more-->

## Purpose

The Dell EMC Streaming Data Platform architecture contains [Pravega](https://www.pravega.io/), an open-source streaming storage system that implements streams and acts as first-class primitive for storing or serving continuous and unbounded data. Pravega [streams](https://pravega.io/docs/latest/pravega-concepts/#streams) are based on an append-only log-data structure that rapidly ingests data into the durable storage. The purpose of this code is to demonstrate ingesting data into a Pravega stream.

## Design
The simplest kind of Pravega application uses a Reader to read from a Stream or a Writer that writes to a Stream. For the Pravega Stream Ingest we will mainly focus on how to write the data to a Pravega Stream on Dell EMC Streaming Data Platform. A simple sample application of both can be found in the [workshop-samples repository](https://github.com/pravega/workshop-samples/tree/master/stream-ingest/src/main/java/com/dellemc/oe/ingest). These samples give a sense on how a Java application could use the [Pravega's Java Client Library](https://pravega.io/docs/v0.1.0/javadoc/clients/index.html) to access Pravega functionality.

The example consists of four applications: `EventWithTimestampWriter`, `EventWriter`, `ImageWritersor`, and `JSONWriter`.  

All applications demonstrate the usage of EventStreamWriter to write an Event to Pravega and have identical ways to write the data to a Pravega stream on Dell EMC SDP. The critical part is in the `run()` method which creates a Stream and outputs the given Event to that Stream.

## Instructions
##### 1. Refer to [this post]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html), which was designed for Flink applications running on Dell EMC Streaming Data Platform. In order to write data to a stream, a scope/project must be created on SDP.

To write the data to a stream on SDP, you need to complete **Step A: Remove `createScope` method from the code** and **Step B. Create Projects** in [this post]({{site.baseurl}}/getting started/2020/07/14/create-flink-project-on-streaming-data-platform.html).

Notice that the **project name** you created will be **same** as the **Pravega scope/namespace** and will be used in the later steps. You should have an Analytics Project showing on your Dashboard as following (project name may vary): 
![streamcut-project]({{site.baseurl}}/assets/heliumjk/images/post/pravega-ingest/streamcut-project.png) 

##### 2. Connect the applications with the Pravega stream on Dell EMC Streaming Data Platform
After setting up the project namespace on SDP, configure Keycloak authentication by following [this post]({{site.baseurl}}/getting started/2020/08/14/Configure-off-cluster-pravega-clients.html). You need to set the same configurations for the `JSONWriter`, `EventWithTimestampWriter`, `EventWriter`, and `ImageWritersor` application.

##### 3. Check the result after running application
The result will print in your terminal or the console if you are using IntelliJ. You will find the Bytes Written are showing up on the Pravega stream dashboard as following. 
![stream-dashboard]({{site.baseurl}}/assets/heliumjk/images/post/pravega-ingest/stream-dashboard.png) 



## Source
[Pravega Workshop Samples](https://github.com/pravega/workshop-samples)


