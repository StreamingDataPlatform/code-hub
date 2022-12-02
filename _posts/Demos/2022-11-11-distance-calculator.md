---
layout: post
category: Demos
tags: [pravega, flink, gateway, mqtt]
subtitle: Dell EMC Streaming Data Analytics - Hands-on
technologies: [SDP]
img: post/distance-calculator-sdp/architecture.png
license: Apache
support: Community
author: 
    name: Luis Liu
    description: I am a nautilus app developer.
    image:
css: 
js: 
---

This post demos how to make a distance calculator on Streaming Data Platform
<!--more-->

## Introduction

- Local deployment of InfluxDB+Grafana+Pravega+IoT GW 
- Optional: Connect Raspberry Pi 4 and run the app to collect the data in json format via a distance sensor
- Using local IDE (Or local Flink cluster) to run Flink Distance Calculation Job
    - Fixed window
        > Calculate the average value of each fixed time window (3s)
    - Distance Calculation logical
        > Normal: <=1m
        > A little Far: > 1m && <= 3m
        > Far: > 3m

### Demo environment
![Demo env]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/architecture.png)

### Workload Flow
1. We fully utilize SDP platform from start to end
2. Collect Distance sensor data via a MQTT Sensor or Use simulator to generate the data from one CSV file and write into one MQTT topic
3. Which further writes it into the Pravega Stream via Pravega Writer API
4. Flink Job read the data from stream using Pravega Reader API
5. Computing and processing in each fixed 3 time window
6. Sink to Influx DB
7. Grafana can Visualize the Output with custom dashboard
![Workflow]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/workflow.png)


### Prerequisites
- Clone [sdp-starter-kit](https://github.com/vangork/sdp-starter-kit)
- [Java 11](https://www.oracle.com/java/technologies/downloads/#java11)
- [Maven 3.6.3](https://archive.apache.org/dist/maven/maven-3/3.6.3/)
- [Helm](https://helm.sh/docs/intro/install/)
- SDP Running in a Cluster
- Flink 1.15.2 runtime

### Setup distance calculator demo using Helm
- The Helm charts for distance calculator demo is in
```
cd sdp-starter-kit/distance-calculator/charts/
```
- Get all available runtime images using the below command.
```
kubectl get runtimeimages -A
```
![setup15]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup15.png)

  Update **flinkImage** variable in **calculator/values.yaml** with the runtime image name available for the flink version 1.15.2. 

- Get **spec.values.image.tag** using below command.
```
kubectl get projectfeature pravegamqttbroker --output="jsonpath={.spec.templates.pravegaMQTTBroker}"
```
![setup16]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup16.png)

  Update **chartVersion** variable in **features/values.yaml** with **spec.values.image.tag**.
- Deploy distance-calculator and mqtt-writer by running below script.
```
./deploy.sh <name_of_the_application>
```
  ex:- ./deploy.sh distance-calculator

  Wait for few minutes until flink application starts
- To visualize dashboard navigate to **Analytics** → **Your Project** → **Features**
- Open **Metrics** endpoint
![setup11]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup11.png)
- Hover on **Dashboard** and Click on **Manage**
![setup12]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup12.png)
- Click on **Distance Calculataor Dashboard**
![setup17]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup17.png)
- Below view can be observed
![setup14]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup14.png)

  **Note:-** In case if you don't see any graph or button to **Zoom to Data**, Apply the time range of 10:37:00 to 11:00:00 for the current date
![setup18]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup18.png)

### Setup distance calculator demo manually

##### 1. Compile distance-calculator project
- Use below command to package and create jar file
    ```
    cd distance-calculator
    mvn clean package
    ```

##### 2. Setup SDP 
- Navigate to **Analytics**
- Click on **New Project**
- Select **Artifact Repository**, **Metrics**, **Pravega MQTT Broker**, **Zookeeper** from the **Features** dropdown
![setup1]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup1.png)
- Click on **Save** button and wait for the project to deploy
- To get **MQTT_BROKER_URL**, **MQTT_USERNAME** and **MQTT_PASSWORD**
- Navigate to **Analytics** → **Your Project** → **Features**
- IP in the "Endpoint" section corresponding to "Pravega MQTT Broker" with port **8883** forms **MQTT_BROKER_URL**. ex: 10.243.52.65:8883
![setup2]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup2.png)
- Click on **pravega-mqtt-broker** in the "Secrets" section to get **MQTT_USERNAME** and **MQTT_PASSWORD**
![setup3]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup3.png)

- Upload Jar file to Artifacts
- Navigate to **Analytics** → **Your Project** → **Artifacts** → **Files**
![setup5]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup5.png)
- Upload the jar file from –> /<Project_Path>/distance-calculator/calculator/target/calculator-1.0.0.jar

##### 3. Create Flink Cluster
- Navigate to **Analytics** → **Your Project** → **Flink** → **Create Flink Cluster**
![setup6]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup6.png)
- Select Flink version 1.15.2 and Click **Save** to create Flink Cluster
![setup7]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup7.png)

##### 4. Run Flink application
- Click **File** 
- Select **Main Application** from dropdown
- Type main class as **io.pravega.flinkapp.DistanceCalculator**
- Select **Flink Version – 1.15.2**
- Click **Create** and wait for the app to Start 
![setup8]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup8.png)

##### 5. Set the Environment variables
- **Note**: -  Edit **Distance.csv** and replace with current dates (To avoid retention policy error-- DATA is only valid for 7 days)
Below Image shows replacing of dates to Today's date
![setup4]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup4.png)
- In **Windows**
    ```
    set MQTT_BROKER_URL=tls://<MQTT_BROKER_URL>:8883
    set MQTT_ALLOW_INSECURE=true
    set MQTT_DATA_FILE=C:\\<Project_Path>\\mqtt-writer\\Distance.csv
    set MQTT_USE_AUTH=true
    set MQTT_USERNAME=<MQTT_USERNAME>
    set MQTT_PASSWORD=<MQTT_PASSWORD>
    ```
- In **Linux/Ubuntu**
    ```
    export MQTT_BROKER_URL=tls://<MQTT_BROKER_URL>:8883
    export MQTT_ALLOW_INSECURE=true
    export MQTT_DATA_FILE=/<Project_Path>/mqtt-writer/Distance.csv
    export MQTT_USE_AUTH=true
    export MQTT_USERNAME=<MQTT_USERNAME>
    export MQTT_PASSWORD=<MQTT_PASSWORD>
    ```

##### 6. Simulate the MQTT writer
- Use below command to run MQTT writer simulator
    ```
    java -jar mqtt-writer\target\mqtt-writer-1.0.0.jar
    ```

##### 7. Get Influxdb Data Source
- Navigate to **Analytics** → **Your Project** → **Features**
![setup9]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup9.png)
- Go to Setting → **Data Source** and  Get Influxdb Data Source name and copy **data_souce_name** (disatanceflink_flnk in below image)
![setup10]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup10.png)
- Edit dashboard.json and replace **datasource** value from ***test_flnk*** to **data_souce_name**

##### 8. Adding Dashboard in Grafana
- Navigate to **Analytics** → **Your Project** → **Features**
- Open **Metrics** endpoint
![setup11]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup11.png)
- Hover on **Dashboard** and Click on **Manage**
![setup12]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup12.png)
- Click on **Import**
![setup13]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup13.png)
- Click **Upload Json File** → Select **dashboard.json**
- Below view can be observed
![setup14]({{site.baseurl}}/assets/heliumjk/images/post/distance-calculator-sdp/setup14.png)

### Source
[https://github.com/vangork/sdp-starter-kit/tree/master/distance-calculator](https://github.com/vangork/sdp-starter-kit/tree/master/distance-calculator)