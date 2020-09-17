---
layout: post
category: "Getting Started"
tags: [flink, java, SDP, Pravega]
subtitle: "Create Projects and Flink Clusters on Dell EMC Streaming Data Platform"
technologies: [SDP, Flink] 
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
##### 1. Remove `createScope` method from the code
**A.** If you have used `createScope` method in Pravega standalone mode. Please **comment out** from your code since the SDP does not allow to create a scope from the code.

##### 2. Create Projects
**A.** Log in to the Dell EMC Streaming Data Platform and click the **Analytics** icon to navigate to the Analytics Projects view which lists the projects you have access to. 
![Analytics-Project]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/analytics-project.png)
**Note:** If you cannot log in to the Dell EMC Streaming Data Platform, please use ```kubectl get ingress -A``` to find all ingress and add to your **/etc/hosts** file.  

**B.** Click **Create Project** button to start with a new project by typing the project name, description and required volume size. This action will also automatically create a scope or namespace for your project in Pravega. 
![Create-Project]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/create-project.png)      


##### 3. Upload Flink artifact to SDP

You must have all your artifact available on the SDP maven repository. There are two different ways for uploading your Flink job artifact:   

###### Option A: Using the Dell EMC Streaming Data Platform UI
**I.** Navigate to **Analytics > Analytics Projects > *project-name* > Artifacts**
![Artifact-UI]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/artifact-ui.png)

**II.** Click **Upload Artifact**. Under the **General** section, specify the **Maven Group**, the **Maven Artifact**, and enter the **Version number**. Under the **Artifact** File section, browse to the JAR file on your local machine and select the file to upload to the repository. 
![Upload-Artifact]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/upload-artifact.png)

###### Option B: Using the Gradle Build Tool
**I.** Make the Maven repo in SDP available to your development workstation. Use `kubectl ingress` command to get the host and IP address for your namespace repo.
```
root@ubuntu:~$ kubectl get ingress -n your-namespace
NAME   HOSTS                                ADDRESS        PORTS   AGE
repo   repo.your-namespace.host.com       11.111.11.11      80     24h
```

Then find and open the `hosts` files which location, depending on the operating system that you are using, is: 
- Windows - `C:\Windows\System32\drivers\etc\hosts`
- Linux - `/etc/hosts`

Use [`vim`](https://www.vim.org/), [`Sublime Text`](https://www.sublimetext.com/), or any other text editors to add the host and IP address information to the `hosts` file.  
After this step, you should find a similar entry from your `hosts` file as following.   
```
root@ubuntu:~$ cat /etc/hosts
    ...                           ...
    ...                           ...  
    ...                           ...          
    ...                           ...
11.111.11.11          repo.your-namespace.host.com
```


**II.** Add a publishing section to your `build.gradle` file. 

The standard `maven-publish` Gradle plugin must be added to the project in addition to a credentials section. The following is an example `build.gradle` file.

Make sure to change the `host` and `protocol` in url to the corresponding information you get from the above step. Depending on the port number, the url protocol is:
- `80` : http
- `443` : https 

Then replace the default `username` and `password` with your credential information.   
You may also need to customize the publication identity, such as **groupId**, **artifactId**, and **version**. If you publish shadow JARs, make sure to add `classifier = ""` and `zip64 true` to the `shadowJar` config. 


```
apply plugin: "maven-publish"
publishing {
    repositories {
        maven {
            url = "http://host/maven2"
            credentials {
            username "johndoe"
            password "**********"
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

**III.** Use the Gradle Extension either from  IntelliJ IDEA or [Command-Line Interface](https://docs.gradle.org/current/userguide/command_line_interface.html) to publish the application JAR file to SDP.


#####  4. Create Flink Clusters and Deploy Applications 

###### Next, you need to create the Flink Clusters, and deploy the applications. There are two ways to complete the above tasks, through the UI on SDP or using [Helm Charts](https://helm.sh/docs/topics/charts/#:~:text=Helm%20uses%20a%20packaging%20format,%2C%20caches%2C%20and%20so%20on). 

#####  <span style="color:#0076ce">Option A: Using Dell EMC Streaming Data Platform UI</span> 
**I.** To create a Flink cluster using the Dell EMC Streaming Data Platform UI, navigate to **Analytics -> *project-name* > Flink Clusters**.  
![Flink-Cluster]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/flink-cluster.png)

**II.** Click **Create Flink Cluster** and complete the cluster configuration fields.
![Create-Flink-Cluster]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/create-flink-cluster.png)

**III.** Navigate to **Analytics > Analytics Projects > *project-name* > Apps**
![App-UI]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/app.png)

**IV.**  Click **Create New App**. Specify the application name, the artifact source, main class, and other configuration information. You also have the chance to create a new Pravega stream for your application.
![Create-App]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/create-app.png)

#####  <span style="color:#0076ce">Option B: Using Helm Chart to deploy the application</span> 

**I.** Install **[Helm CLI](https://helm.sh/docs/intro/install/)** into your local development environment.

Follow [this guide](https://helm.sh/docs/intro/install/) to install Helm. It can be installed either from source, or from pre-built binary releases.

**II.** Generate the **[Helm Chart](https://helm.sh/docs/topics/charts/)** file structure  
To use Helm Chart, you need to have a collection of files inside a directory. The directory should be named as `charts`. Then refer to [this github repo](https://github.com/pravega/workshop-samples/tree/master/charts), the [Helm Charts documentation](https://helm.sh/docs/topics/charts/), and the following structure to organize your Chart file.  

```
charts/               # A directory containing any charts upon which this chart depends
    AplicationName/         # This structure will allow one application per cluster
        Chart.yaml          # A YAML file containing information about the chart
        values.yaml         # The default configuration values for this chart
        templates/          # A directory of templates that, when combined with values,
                            # will generate valid Kubernetes manifest files.
```
**III.** Add **FlinkCluster** and **application** `yaml` files to the `template` folder.  
Next, you need to create template `yaml` files to generate manifest files on the Kubernetes cluster, combining with the configuration in `values.yaml`.

The template folder in [workshop-sample repo](https://github.com/pravega/workshop-samples/tree/master/charts/templates) will give you a general idea for developing your template file. We highly recommend one application per cluster. Once the application has been developed and tested in isolation, advanced users can optimize resources by sharing clusters. Your chart file structure should look similar to the following when using one application per cluster.

```
charts/              
    TestApplication/
        Chart.yaml          
        values.yaml         
        templates/          
            FlinkCluster.yaml               # Template file for Flink Cluster
            FlinkApplication.yaml           # Template file for your application. 
    ...
    ...                                     # You can have multiple applications.
    ...                     
```

**IV.** Go to your project home directory  
Since all chart files have been organized into a structure under the `charts` folder, you need to visit the parent directory of `charts` folder in order to install or upgrade the Helm Chart to SDP. 

Take the following structure as an example; after this step, you should locate inside the `FlinkApp` folder.

```
FlinkApp/
    charts/              
        TestApplication/
            Chart.yaml          
            values.yaml         
            templates/          
                FlinkCluster.yaml               # Template file for Flink Cluster
                FlinkApplication.yaml           # Template file for your application. 
        ...
        ...                                     # You can have multiple applications.
        ...   
    TestApplication/                 
```

**V.**  Using **[Helm CLI](https://helm.sh/docs/intro/quickstart/)** to install or upgrade the charts  

Open the terminal window from the project home directory that you access from the above step. Then you can use [`helm upgrade`](https://helm.sh/docs/helm/helm_upgrade/) command to deploy your application and charts to SDP. 

The following are the command examples that will create a release name `jsonreader`. The benefit of using  [`helm upgrade`](https://helm.sh/docs/helm/helm_upgrade/) is that if a release by this name doesn't already exist, it will automatically run an install; otherwise, it will upgrade a release to a new version of a chart.


```
helm upgrade --install --timeout 600s jsonreader --wait --namespace workshop-samples charts
```


##### 5. Check Project Status
**A.** Navigate to **Analytics > Analytics Projects > *project-name***. Use the links at the top of the project dashboard to view Apache Flink clusters, applications, and artifacts associated with the project.
![Dashboard-UI]({{site.baseurl}}/assets/heliumjk/images/post/flink-sdp-setup/dashboard.png)


## Source
[Dell EMC Streaming Data Platform InfoHub](https://www.dell.com/support/article/en-us/sln319974/dell-emc-streaming-data-platform-infohub?lang=en)

## Documentation
For the detailed description for each attribute on the configuration screen, please refer to the  [Dell EMC Streaming Data Platform Developer's Guide](https://www.dellemc.com/en-us/collaterals/unauth/technical-guides-support-information/2020/01/docu96951.pdf).
