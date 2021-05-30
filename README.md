# The Google Cloud Dataproc Bigdata Ecosystem Cookbook

How to build up from scratch a Bigdata Ecosystem using Dataproc Clusters supplied by Google Cloud Platform (GCP). Through Docker and Zeppelin we will build up a system with nifi, kafka, elasticsearch,kibana and zeppelin notebooks.

We will use Santander Smart City data to ingest into our system and play around

Some of the contents related to kafka ( specially connectivity with conteinerized-kafka are taken from this web page [Why Can´t I Connect to Kafka](https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/) written by Robin Moffat). I believe his article is a remarkable explanation about how to connect to kafka and cannot be further improved so I borrowed his own code and diagrams in order to explain how to make a connection to kafka and what could possibly go wrong.


Introduction
============

## Contents

- [What is this Cookbook](README.md#what-is-this-cookbook)
- [Building up the Dataproc Cluster](README.md#building-up-the-architecture-the-dataproc-cluster)
  - [What is Google Cloud Dataproc](README.md#what-is-google-cloud-dataproc)
  - [First Steps](README.md#first-steps)
  - [Connecting to Apache Kafka](README.md#Connecting-to-kafka)
- [Putting all agents working together](README.md#putting-all-agents-working-together)
  - [Using Apache nifi](README.md#nifi)
  - [Using Apache kafka](README.md#kafka)
 
## What is this Cookbook

This book is intended to be a proof of concept to spin up a fully bigdata ecosystem cluster from scratch using GCP.

We will suck data from online real data from the city of Santander, available to us through the Smart City web page, and ingest it into our system, which will be a Google Cloud Dataproc cluster, provisioned with nifi, kafka and elasticsearch Docker containers. The visual presentation will be done using kibana.

At the same time we will be able to interact with the system in real time with zeppelin notebooks.

In this cookbook we will cover all Data Engeneering platform key areas (Connect, Buffer, Processing Framework, Store, Visualize)

We will split the cookbook into two main parts:

**First**: Building up the architecture

**Second**: Putting all main agents ( nifi, elasticsearch, kibana, zeppelin) working together to ingest and extract meaning from the traffic data of the city of Santander

# Building up the architecture: The Dataproc Cluster

### What is Google Cloud Dataproc 

Datapro is bla bla bla
Where to get it.
300 $ to play around

![Cloud Components](/images/00-Dataproc.png)

### Architecture
- Interaction between master node (local in the image) with Dockerized Kafka broker container



### First Steps

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

- Create [docker-compose.yml](/scripts/docker-compose.yml) 

- Spin up all containers defined within docker-compose.yml: nifi, elasticsearch, zookeeper, kafka and kibana.
   - **Nifi container** spins up the apache/nifi:latest image, which allows us to use a powerful and reliable system to process and distribute data, the  [Apache Nifi project](http://http://nifi.apache.org/download.html). It will be used as a single point of data entrace into the system, allowing us to ingest and transform data in a visual way. After that, data will be sinked to different locations such as hdfs, kafka brokers and mysql database.
   - **Zookeeper container**
 confluentinc/cp-zookeeper:5.5.0
   - **Kafka broker container**
 confluentinc/cp-kafka:5.5.0
   - **Elasticsearch container**
 docker.elastic.co/elasticsearch/elasticsearch:7.6.2
   - **Kibana container**
 docker.elastic.co/kibana/kibana:7.6.2
      

        	docker-compose up -d 

The first time we issue this command will download nifi, zookeeper, kafka, elasticsearch an kibana images from Docker Hub, as we don't have it yet.
We can use the detached mode to avoid the logs printed out to the screen.

![docker-compose up -d  exit](/images/00_docker-compose-up.png)

Verify all 5 containers are up and none of them exited:

    docker ps -a

![docker-compose up -d  exit](/images/10_docker-compose-up.png)


- Create a SSH tunnel to connect to nifi and kibana.

Through docker-compose.yml we have exposed port 8080 (from nifi container) to 8060 in localhost ( our vm instance which acts as master node). So we have to tunnel from our local machine ( laptop ) to the VM port 8060 to get access to Nifi Web UI. One way of doing it is by manually using PuttY ,generating ssh-keys and installing them in the VM, connecting to the VM and specifying tunnel port in Putty. One easier way is through Google Cloud Standard Development Kit keeping in mind that you have to have Web browser Chrome already installed.

Windows as I have, I downloaded [Google Cloud Client](https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe)  and installed it.
Once installed their features are embedded within cmd or powershell commands and you can use gcloud extensions as if they belong to Windows cmd itself.

In Google Cloud there is a cheat sheet for Windows, Mac and Linux Systems.

![SSH tunnel](/images/00_gcloud.png)


- Open a ssh tunnel through port 1080

     	gcloud compute ssh cluster-uoc-m ^
       	 --project=data-lakes-313014 ^
       	 --zone=europe-west6-b -- -D 1080 -N

This command will open a ssh tunnel through Putty generating ssh keys for you. That's why PuttY opens and is waiting for you to add VMs public key to Puttys cache and carry on connecting. 

![SSH tunnel](/images/10_gcloud.png)

- Launch chrome through ssh tunnel to access desired port (In our case Nifi stands for localhost:8060 which is the port exposed to the VM)

		C:\%ProgramFiles(x86)%\Google\Chrome\Application\chrome.exe ^
  		--proxy-server="socks5://localhost:1080" ^
  		--user-data-dir="%Temp%\cluster-uoc-m" http://cluster-uoc-m:8060

If you don't have the environment variable %ProgramFiles(x86)% set up to the actual path you can navigate to where chrome is an execute just the bit from chrome.exe onwards.

Then, a chrome window will pop up with the call made before (http://cluster-uoc-m:8060) and a nifi welcome page will show up redirecting to the main page after a while. 

Nifi is ready to go!


![Apache nifi](/images/00_nifi.png)

## Connecting to Kafka

As kafka is up and running let's go and try create a topic within kafka broker container to check everything is fine. In order to do so let's have a look to the kafka arquitecture that we have built up through [docker-compose.yml](/scripts/docker-compose.yml)


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

Check out that there are two advertised listerners on broker:9092 and localhost:19092. The whole thing of kafka connection boils down to these two listeners.
Why using two instead of the default one? The reason being that in order to establish a successful connection to kafka two things must succeed:

1. The initial connection to a broker (the bootstrap) which returns metadata to the client, including a list of all the brokers in the cluster and their connection endpoints.
2. The client then connects to one (or more) of the brokers returned in the first step as required. If the broker has not been configured correctly, the connections will fail.

What usually happens is that you often only care for the first connection. Once it is succesful you say Aha! I'm connected! But you're not. What often happens is described in the graph below:

![Can't resolve host](/images/20_kafka_configuration.png)


If you expose port 9092 as you normally do, the first connection against kafka (the one that gets the metadata) should succeed, but the second one should fail.
	…
	  broker:
	    image: confluentinc/cp-kafka:5.5.0
	    container_name: broker
	    networks: 
	      - rmoff_kafka
	    ports:
	      - "9092:9092"
	    environment:
	    KAFKA_BROKER_ID: 1
	    KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
	    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:9092
	    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1 
	…

The client (for example a Zeppelin notebook which is running on the master node of the VM ) cannot resolve  the advertised listener broker:9092 as it has no means to resolve the address.

Once you use more than one, you are bound to procure the security protocol as well. Hence why we write down PLAINTEXT security protocol twice.

To prove that is up and listenging properly we will check that:
1. There is connectivity among any containers from the same network (Spin up a docker container with some python code that interacts with kafka) 
2. There is connectivity from local (master node within VM) to Kafka. (Launch spark-shell from the VM an run some scala code to produce/consume to kafka broker) 

All of the ideas and code are from retrieved from this amazing page written by .... All the merit goes to him!

As mention before we will use this arquitechture to connect to kafka broker. This diagram from ... to 
![Architecture](/images/10_kafka_configuration.png)
Source: https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/



The behaviour to connect to a Dockerized kafka broker is this:



### Prove there is connection to kafka broker container from another container

To do that we will spin up a container based on an image named python_kafka_test_client from a Dockerfile. This file has a python3 base image. We will add a python program python_kafka_test_client.py that tries to connect to kafka and produce and consume some test messages from and to the topic "topic_test".

- Python test connection program ![python_kafka_test_client.py](https://github.com/ajimeneo/dataproc-bigdata/blob/main/scripts/python_kafka_test_client.py). Use nano or other suitable text editor to create the python_kafka_test_client.py file using the available source.


- Create a ![Dockerfile](https://github.com/ajimeneo/dataproc-bigdata/blob/main/scripts/Dockerfile). We will use this Dockerfile to create an image and spin up a container from it.

	docker build -t python_kafka_test_client .
	
List images to check out that python_kafka_test_client image has been created

	docker images 

 ![Images list](/images/10_docker_build.png)
 
- Spin up a Docker container with python_kafka_test_client image

		docker run --network=nifinet --rm --name python_kafka_test_client \
        	--tty python_kafka_test_client broker:9092

The program expects a host as a first param. As you can see from the image broker:9092 is the expected path.


 ![Success](/images/00_connection_successful.png)

- Launch a spark-shell and run a scala program that connects to kafka broker	

# Putting all agents working together

## Using Apache Nifi
		
| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/

