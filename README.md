# The Google Cloud Dataproc Bigdata Ecosystem Cookbook
How to build up a bigdata ecosystem using nifi, kafka, elasticsearch and kibana with zeppelin notebooks using a google cloud dataproc clusterfrom scratch

![Cloud Components](/images/00-Dataproc.png)

### Data Engineer

Introduction
============

## Contents

- [What is this Cookbook](README.md#what-is-this-cookbook)
- [Building the Dataproc Cluster](README.md#building-the-dataproc-cluster)
  - [What is Google Cloud Dataproc](README.md#what-is-google-cloud-dataproc)
  - [Firs Steps](README.md#first-steps)
- [Why Big Data](03-AdvancedSkills.md#why-big-data)
    - [Planning is Everything](03-AdvancedSkills.md#planning-is-everything)


## What is this Cookbook

This book is intended to be a proof of concept to spin up a fully bigdata ecosystem cluster from scratch. 

We will suck data from online real data from the city of Santander, available to us through the Smart City web page, and pour it into our system, which will be a Google Cloud Dataproc cluster, provisioned with nifi,kafka and elasticsearch Docker containers. The visual presentation will be done using kibana.

At the same time we will be able to interact with the system in real time with zeppelin notebooks.

In this cookbook we will cover all Data Engeneering platform key areas (Connect, Buffer, Processing Framework, Store, Visualize)

**Help make this book awesome!**
If you have some cool links or topics for the cookbook, please become a
contributor on GitHub: <https://github.com/andkret/Cookbook>. Fork the

## Building the Dataproc Cluster

### What is Google Cloud Dataproc 

Datapro is bla bla bla
Where to get it.
300 $ to play around

### Architecture
- Interaction between master node (local in the image) with Dockerized Kafka broker container

![Architecture](/images/10_kafka_configuration.png)

### First Steps

#### Spining up a 2 Worker Cluster Data Node provisioned with Docker and zeppeling notebooks.

- Provision the master node with Docker Compose
  
	- Download the Docker Compose binary into the /usr/local/bin directory with wget or curl :
    
            sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" \
    		-o /usr/local/bin/docker-compose
    
	- Use chmod to make the Compose binary executable:
  
      		sudo chmod +x /usr/local/bin/docker-compose
  
	- To verify the installation, use the following command which prints the Compose version:
  
      		docker-compose --version
    
	- The output will look something like this:  
    
      		docker-compose version 1.23.1, build b02f1306

- Create nifinet network

		docker network create nifinet --driver bridge	

- Create docker-compose.yml

 This docker-compose file spins up 5 different containers:
 - **Nifi container**
	It contains the [Apache Nifi](http://http://nifi.apache.org/download.html) project, which is a a powerful and reliable system to process and distribute data. It will be used as a single point of data entrace into the system, allowing us to ingest and transform data. After that, data will be sinked to different destinations sucha as hdfs distributed file system, mysql database and kafka brokers.
	The image we use within docker-compose.yml is **apache/nifi:latest**
 - **Zookeeper container**
 confluentinc/cp-zookeeper:5.5.0
 - **Kafka broker container**
 confluentinc/cp-kafka:5.5.0
 - **Elasticsearch container**
 docker.elastic.co/elasticsearch/elasticsearch:7.6.2
 - **Kibana container**
 docker.elastic.co/kibana/kibana:7.6.2
      
Start all 5 containers:

    docker-compose up -d 

The first time we issue this command will download nifi, zookeeper, kafka, elasticsearch an kibana images from Docker Hub, as we don't have it yet.
We can use the detached mode to avoid the logs printed out to the screen.

![docker-compose up -d  exit](/images/00_docker-compose-up.png)

Verify all 5 containers are up and none of them exited:

    docker ps -a

![docker-compose up -d  exit](/images/10_docker-compose-up.png)


- Create a SSH tunnel to connect to nifi and kibana.

Through docker-compose.yml we have exposed port 8080 (from nifi container) to 8060 in localhost ( our vm instance which acts as master node). So we have to tunnel from our local machine ( laptop ) to the VM port 8060 to get access to Nifi Web UI. One way of doing it is by manually using PuttY ,generating ssh-keys and installing them in the VM, connecting to the VM and specifying tunnel port in Putty. One easier way is through Google Cloud client.

In my case, as is a Windows, [Google Cloud Client] (https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe)  




| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/

