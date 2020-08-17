---
layout: post
category: "Getting Started"
tags: [flink, java, SDP, Pravega]
subtitle: "Configure off-cluster Pravega clients on Dell EMC Streaming Data Platform"
img: pravega-client.png
license: Apache
support: Community
author: 
    name: Youmin Han
    description: Nautilus App Developer
    image: batman.png
css: 
js: 
---
This is a general guide for configuring Pravega applications/clients to connect to SDP when the client is running off-cluster.
<!--more-->

## Purpose
Before ingesting data into a Pravega stream on Dell EMC Streaming Data Platform, you need to configure off-cluster Pravega clients for the secruity verify. The purpose of this guide is to demonstrate configuring keycloak authentication, retrieving Pravega controller external IP, and setting environment variables.

## Instructions
#### 1. Configure keycloak authentication
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

#### 2. Retrieve Pravega controller external IP
In order to access the Pravega cluster from Dell EMC Streaming Data Platform, you need to connect with Pravega controller. Use `kubectl` to get the `EXTERNAL-IP` of the nautilus-pravega-controller service.
```
kubectl get service nautilus-pravega-controller -n nautilus-pravega

NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                          AGE
nautilus-pravega-controller   LoadBalancer   10.100.200.243   11.111.11.111   10080:32217/TCP,9090:30808/TCP   6d5h
```

#### 3. Retrieve Pravega controller external IP
Before running the application, set the following environment variables. This can be done by setting the IntelliJ run configurations. (Go to run -> Edit Configurations -> Select your application -> Environment Variables -> click Browse icon. Fill the details mentioned below screen. Add all below program environment variables.) Make sure to change the `PRAVEGA_CONTROLLER` to the appropriate `EXTERNAL-IP` address. Notice that the assigned value for `PRAVEGA_SCOPE` and `PRAVEGA_STREAM` may need to be changed based on the settings when you created the Flink project and application on SDP.   
```
pravega_client_auth_method=Bearer
pravega_client_auth_loadDynamic=true
KEYCLOAK_SERVICE_ACCOUNT_FILE=${HOME}/keycloak.json
PRAVEGA_CONTROLLER=tcp://<pravega controller external-ip>:9090
PRAVEGA_SCOPE=<pravega scope>
PRAVEGA_STREAM=<target stream name>
```

In addition, if your have a reader running on Dell EMC SDP at the same time, please make sure to pass the same Pravega scope and stream parameters. More details are discussed in the [workshop-samples README](https://github.com/pravega/workshop-samples#running-jsonwriter-from-intelij).     

Then you can save the configuration and run a Pravega stream ingest application. You can find an example application from the post for [Pravega Stream Ingest on Dell EMC Streaming Data Platform]({{site.baseurl}}/getting started/2020/08/04/pravega-stream-ingest-on-streaming-data-platform.html).


## Source
[Workshop Samples Repository](https://github.com/pravega/workshop-samples)

