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

- Provisioning Docker Compose
  
	- Download the Docker Compose binary into the /usr/local/bin directory with wget or curl :
    
            sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" \
    		-o /usr/local/bin/docker-compose
    
	- Use chmod to make the Compose binary executable:
  
      		sudo chmod +x /usr/local/bin/docker-compose
  
	- To verify the installation, use the following command which prints the Compose version:
  
      		docker-compose --version
    
	- The output will look something like this:  
    
      		docker-compose version 1.23.1, build b02f1306

- Creating nifi-tier network
	docker network create nifi-tier --driver bridge	

- Creating docker-compose.yml


version: '3.5'   
    services:
      zookeeper:
        image: confluentinc/cp-zookeeper:5.5.0
        container_name: zookeeper
        networks: 
          - nifinet
        environment:
          ZOOKEEPER_CLIENT_PORT: 2181
    
      broker:
        image: confluentinc/cp-kafka:5.5.0
        container_name: broker
        networks: 
          - nifinet
        ports:
          - "19092:19092"
        depends_on:
          - zookeeper
        environment:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:9092,CONNECTIONS_FROM_HOST://localhost:19092
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,CONNECTIONS_FROM_HOST:PLAINTEXT
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    
      nifi:
        image: apache/nifi
        container_name: nifi
        networks: 
          - nifinet
        ports:
    	  - "8060:8080"
    	volumes:
    	  - /etc/hadoop/conf:/hadoop-conf	
        depends_on:
          - kafka
    
      elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
        container_name: elasticsearch
        networks: 
          - nifinet
        ports:
    	  - "9200:9200"
    	  - "9300:9300"
        depends_on:
          - nifi
        environment:
          discovery.type: single-node		    
    
      kibana:
        image: docker.elastic.co/kibana/kibana:7.6.2
        container_name: kibana
        networks: 
          - nifinet
        ports:
    	  - "5601:5601"
        depends_on:
      - elasticsearch
      
#### KAFKA Commands

Start Zookeeper container for Kafka:

    docker run -d --name zookeeper-server   \
        --network app-tier   \
        -e ALLOW_ANONYMOUS_LOGIN=yes    \
        bitnami/zookeeper:latest

Start Kafka container:

    docker run -d --name kafka-server  \
        --network app-tier  \
        -e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper-server:2181  \
        -e ALLOW_PLAINTEXT_LISTENER=yes  \
        bitnami/kafka:latest



| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)


My YouTube video how to set up Kafka at home:
<https://youtu.be/7F9tBwTUSeY>

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/

