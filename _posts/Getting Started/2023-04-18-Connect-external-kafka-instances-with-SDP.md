---
layout: post
category: "Getting Started"
tags: [flink, spark, java, python, sdp, kafka]
subtitle: "Connecting external Kafka instances with Dell EMC Streaming Data Platform"
technologies: [SDP, Kafka, Flink, Spark]
img: flink-sdp-setup/sdp.jpg
license: Apache
support: Community
author: 
    name: Divya Mahajan
    description: Nautilus App Developer
    image: mascot.png

css: 
js: 
---
This is a general guide for connecting external Kafka instances with Dell EMC Streaming Data Platform.
<!--more-->

## Purpose

The purpose of this overview and sample code is to demonstrate how Streaming Data Platform(SDP) administrators and users can work with external Kafka instances to implement a typical use case utilizing most of SDP streaming capabilities (i.e. Spark and Flink jobs). 

## Design
The simplest kind of application to showcase this uses a reader to read from a Kafka Stream and a writer that writes to another Kafka Stream using Flink and Spark applications created on Dell EMC Streaming Data Platform. A sample application for both can be found in the [sdp-kafka-streaming repository](https://github.com/StreamingDataPlatform/sdp-kafka-streaming). These samples give a sense on how we can create Flink and Spark applications on Dell EMC Streaming Data Platform that connect to external Kafka instances.

The example consists of two applications: `SdpKafkaStreamingUsingFlink` and `SdpKafkaStreamingUsingSpark`.  

Both applications demonstrate the usage of Kafka streams to read and write data from Dell EMC Streaming Data Platform using Flink and Spark applications respectively. These examples assume that the user already has an external Kafka instance available and running which can be used to connect to SDP using flink and spark applications.

## Instructions
##### 1. In order to read/write data to a kafka stream, an Analytics Project must be created on SDP as following (project name may vary): 
![Analytics-Project]({{site.baseurl}}/assets/images/posts/sdp-kafka/sdp-project.png)


##### 2. Create the application using Flink/spark jobs on Dell EMC Streaming Data Platform that connects to external Kafka instance to read and write data streams. For more detailed instructions, please refer [sdp-kafka-streaming repository](https://github.com/StreamingDataPlatform/sdp-kafka-streaming).


## Source
[SDP Kafka Samples](https://github.com/StreamingDataPlatform/sdp-kafka-streaming)


