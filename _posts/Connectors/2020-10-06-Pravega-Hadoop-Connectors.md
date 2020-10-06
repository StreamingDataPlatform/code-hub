---
layout: post
category: Connectors
tags: [pravega, hadoop, connector]
subtitle: Enable Apache Hadoop to read and write Pravega streams
technologies: [Pravega, Hadoop]
img: hadoop.png
license: Apache
support: Community
author: 
    name: Youmin Han
    description: Nautilus App Developer
    image: batman.png
css: 
js: 
---
This post introduces connectors to read and write [Pravega](http://pravega.io/) Streams with [Apache Hadoop](https://hadoop.apache.org/) stream processing framework.
<!--more-->

The connectors can be used to build end-to-end stream processing pipelines (see [Samples](https://github.com/pravega/pravega-samples))  that use Pravega as the stream storage and message bus, and Apache Hadoop for computation over the streams.

## Purpose
Implements both the input and the output format interfaces for Hadoop. It leverages Pravega batch client to read existing events in parallel; and uses write API to write events to Pravega stream.

## Build and Usage
The build script handles Pravega as a _source dependency_, meaning that the connector is linked to a specific commit of Pravega (as opposed to a specific release version) in order to facilitate co-development. This is accomplished with a combination of a _git submodule_ and the use of Gradle's _composite build_ feature. 

More details about build and usage are showing in the README of the [hadoop connector](https://github.com/pravega/hadoop-connectors#cloning-the-repository)

## Pravega Hadoop Connector Examples
You can find the Pravega Hadoop connector examples from the [pravega-samples repo](https://github.com/pravega/pravega-samples/tree/master/hadoop-connector-examples). There are two code examples to give you some basic ideas on how  to use hadoop-connectors for Pravega.   
- **Wordcount:** Counts the words from a Pravega `Stream` filled with random text to demonstrate the usage of Hadoop connector for Pravega.
- **Terasort:** Sort events from an input Pravega `Stream` and then write sorted events to one or more streams.

Before running the application, you may also need to set up the following Environment Prerequisites:  
**1.** Pravega running (see [here](http://pravega.io/docs/latest/getting-started/) for instructions)  
**2.** Build [pravega-samples](https://github.com/pravega/pravega-samples#build-instructions) repository  
**3.** Setup and start HDFS (refer [this document](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html) from Apache Hadoop)    

## Source
- Hadoop Connectors: [https://github.com/pravega/hadoop-connectors](https://github.com/pravega/hadoop-connectors)
- Hadoop Connectors Examples: [https://github.com/pravega/pravega-samples/tree/master/hadoop-connector-examples](https://github.com/pravega/pravega-samples/tree/master/hadoop-connector-examples)

## Documentation
To learn more about how to build and use the Flink Connector library, follow the connector documentation [here](http://pravega.io/).

More examples on how to use the connectors with Flink application can be found in [Pravega Samples](https://github.com/pravega/pravega-samples) repository.

## License
Flink connectors for Pravega is 100% open source and community-driven. All components are available
under [Apache 2 License](https://www.apache.org/licenses/LICENSE-2.0.html) on GitHub.