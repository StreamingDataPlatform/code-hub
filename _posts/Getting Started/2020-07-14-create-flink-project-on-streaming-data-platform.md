---
layout: post
category: "Getting Started"
tags: [flink, java, SDP, Pravega]
subtitle: "Create Projects and Flink Clusters on Dell EMC Streaming Data Platform"
img: sdp.jpg
license: Apache
support: Community
author: 
    name: Youmin Han
    description: Nautilus App Developer
    image: batman.png
css: 
js: 
---
This is a general guide for creating and setting up Flink job on Dell EMC Streaming Data Platform.
<!--more-->

## Purpose
Apache Flink is the embedded analytics engine in the Dell EMC Streaming Data Platform which provides a seamless way for development teams to deploy analytics applications. This post describes the general instruction of using Apache Flink<sup>®</sup> applications with the Streaming Data Platform.

## Instructions
##### A. Remove `createScope` method from the code
**1.** If you have used `createScope` method in Pravega standalone mode. Please **comment out** from your code since the SDP does not allow to create a scope from the code.

##### B. Create Projects
**1.** Log in to the Dell EMC Streaming Data Platform and click the **Analytics** icon to navigate to the Analytics Projects view which lists the projects you have access to. 
![Analytics-Project]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/analytics-project.png)
**Note:** If you cannot log in to the Dell EMC Streaming Data Platform, please use ```kubectl get ingress -A``` to find all ingress and add to your **/etc/hosts** file.  

**2.** Click **Create Project** button to start with a new project by typing the project name, description and required volume size. This action will also automatically create a scope or namespace for your project in Pravega. 
![Create-Project]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/create-project.png)      


##### C. Upload artifact to SDP
###### Option I: Using the Dell EMC Streaming Data Platform UI
**1.** Navigate to **Analytics > Analytics Projects > *project-name* > Artifacts**
![Artifact-UI]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/artifact-ui.png)

**2.** Click **Upload Artifact**. Under the **General** section, specify the **Maven Group**, the **Maven Artifact**, and enter the **Version number**. Under the **Artifact** File section, browse to the JAR file on your local machine and select the file to upload to the repository. 
![Upload-Artifact]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/upload-artifact.png)

###### Option II: Using the Gradle Build Tool
**1.** Make the Maven repo in SDP available to your development workstation.
```
kubectl port-forward service/repo 9090:80 --namespace project-name
```

**2.** Add a publishing section to your `build.gradle` file. The standard `maven-publish` Gradle plugin must be added to the project in addition to a credentials section. You may also need to customize the publication identity, such as **groupId**, **artifactId**, and **version**. If you publish shadow JARs, make sure to add `classifier = ""` and `zip64 true` to the `shadowJar` config. The following is an example `build.gradle` file.
```
apply plugin: "maven-publish"
publishing {
    repositories {
        maven {
            url = "http://localhost:9090/maven2"
            credentials {
            username "desdp"
            password "password"
            }
            authentication {
                basic(BasicAuthentication)
            }
        }
    }

    publications {
        maven(MavenPublication) {
            groupId = 'com.dell.com'
            artifactId = 'anomaly-detection'
            version = '1.0'
            from components.java
        }
    }
}
```

**3.** Use the Gradle Extension either from  IntelliJ IDEA or [Command-Line Interface](https://docs.gradle.org/current/userguide/command_line_interface.html) to publish the application JAR file to SDP.


##### Next, you need to create the Flink Clusters, and deploy the applications. There are two ways to complete the above tasks, through the UI on SDP or using [Helm Charts](https://helm.sh/docs/topics/charts/#:~:text=Helm%20uses%20a%20packaging%20format,%2C%20caches%2C%20and%20so%20on). 

####  <span style="color:#0076ce">Option A: Using Dell EMC Streaming Data Platform UI</span> 

#####  D. Create Flink Clusters
**1.** To create a Flink cluster using the Dell EMC Streaming Data Platform UI, navigate to **Analytics -> *project-name* > Flink Clusters**.  
![Flink-Cluster]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/flink-cluster.png)

**2.** Click **Create Flink Cluster** and complete the cluster configuration fields.
![Create-Flink-Cluster]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/create-flink-cluster.png)



##### E. Deploy Applications
**1.** Navigate to **Analytics > Analytics Projects > *project-name* > Apps**
![App-UI]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/app.png)

**2.**  Click **Create New App**. Specify the application name, the artifact source, main class, and other configuration information. You also have the chance to create a new Pravega stream for your application.
![Create-App]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/create-app.png)

####  <span style="color:#0076ce">Option B: Using Helm Chart to deploy the application</span> 

#####  D. Create the Chart File Structure and add `yaml` files to the template

**1.** Install **[Helm CLI](https://helm.sh/docs/intro/install/)** into your local development environment.

Follow [this guide](https://helm.sh/docs/intro/install/) to install Helm. It can be installed either from source, or from pre-built binary releases.

**2.** Generate the **[Helm Chart](https://helm.sh/docs/topics/charts/)** file structure  
To use Helm Chart, you need to have a collection of files inside a directory. The directory should be named as `charts`. Then refer to [this github repo](https://github.com/pravega/workshop-samples/tree/master/charts), the [Helm Charts documentation](https://helm.sh/docs/topics/charts/), and the following structure to organize your Chart file.  

```
charts/               # A directory containing any charts upon which this chart depends
  Chart.yaml          # A YAML file containing information about the chart
  values.yaml         # The default configuration values for this chart
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
```
**3.** Add **FlinkCluster** and **application** `yaml` files to the `template` folder.  
Next, you need to create template `yaml` files to generate manifest files on the Kubernetes cluster, combining with the configuration in `values.yaml`.

The template folder in [workshop-sample repo](https://github.com/pravega/workshop-samples/tree/master/charts/templates) will give you a general idea for developing your template file. Your chart file structure should look similar to the following way.

```
charts/              
  Chart.yaml          
  values.yaml         
  templates/          
    FlinkCluster.yaml               # Template file for Flink Cluste
    YourApplication.yaml            # Template file for your application. It can be multiple applications.
            ...
            ...
            ...                    
```

##### E. Deploy Applications
**1.** Go to your project home directory  
Since all chart files have been organized into a structure  under the `Charts` folder, you need to visit the parent directory of `Chart` folder in order to install or upgrade the Helm Chart to SDP. 

Take the following structure as an example; after this step, you should locate inside the `FlinkApp` folder.

```
FlinkApp/
    charts/              
        Chart.yaml          
        values.yaml         
        templates/          
            FlinkCluster.yaml               # Template file for Flink Cluste
            YourApplication.yaml            # Template file for your application. It can be multiple applications.
                    ...
                    ...
                    ...   
    FlinkApplication/                 
```

**2.**  Using **[Helm CLI](https://helm.sh/docs/intro/quickstart/)** to install or upgrade the charts  

Open the terminal window from the project home directory that you access from the above step. Then you can use either [`helm install`](https://helm.sh/docs/helm/helm_install/) or [`helm upgrade`](https://helm.sh/docs/helm/helm_upgrade/) command to deploy your application and charts to SDP. 

The following are the command examples that will create a release name `jsonreader`. The benefit of using  [`helm upgrade`](https://helm.sh/docs/helm/helm_upgrade/) is that if a release by this name doesn't already exist, it will automatically run an install; otherwise, it will upgrade a release to a new version of a chart.


```
helm install --timeout 600s jsonreader --wait --namespace workshop-samples charts

// Or

helm upgrade --install --timeout 600s jsonreader --wait --namespace workshop-samples charts
```


##### F. Check Project Status
**1.** Navigate to **Analytics > Analytics Projects > *project-name***. Use the links at the top of the project dashboard to view Apache Flink clusters, applications, and artifacts associated with the project.
![Dashboard-UI]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/dashboard.png)


## Source
[Dell EMC Streaming Data Platform InfoHub](https://www.dell.com/support/article/en-us/sln319974/dell-emc-streaming-data-platform-infohub?lang=en)

## Documentation
For the detailed description for each attribute on the configuration screen, please refer to the  [Dell EMC Streaming Data Platform Developer's Guide](https://www.dellemc.com/en-us/collaterals/unauth/technical-guides-support-information/2020/01/docu96951.pdf).
