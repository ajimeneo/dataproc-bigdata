# Building up the architecture: The Dataproc Cluster

## Contents

  - [Setting up the cluster](01-BuildingUp.md#Setting-up-the-cluster)
  - [Troubleshooting Common Issues with Kafka](01-BuildingUp.md#Troubleshooting-common-issues-with-kafka)
  - [Setting Up Zeppelin](01-BuildingUp.md#setting-up-Zeppelin)

## Setting up the cluster

Once you set up the account, create a new Project, and search for the Dataproc section.

![Cloud Components](/images/00-Dataproc.png)

Then hit Create a Cluster and a window will pop up. Fill in the blanks with an appropriate cluster name, location (in my case [europe-west6](https://cloud.google.com/compute/docs/regions-zones)) and cluster type. If you want to save as much credit as you can, choose **Single Node**, buy if you want to recreate a real cluster then choose a **Standard One**.

![Cloud Components](/images/01-Dataproc.png)

 Image type and version should be ok with the default **2.0-debian10**. 

![Cloud Components](/images/02-Dataproc.png)

Enable component gateaway to get access to the web Interfaces and select Docker and Zeppelin as Optional components as all the Big Data tools we're going to use are Dockerized containers. Zeppelin notebooks comes in handy to interact with the cluster.

![Cloud Components](/images/03-Dataproc.png)

Then choose a VM instance of your liking for the Master node:

![Cloud Components](/images/05-Dataproc.png)

And the workers (2 nodes which is the minimum default. You cannot choose less than that):

![Cloud Components](/images/06-Dataproc.png)

You can do all the steps done before in a programmatically way using Google SDF as follows:

![Cloud Components](/images/04-Dataproc.png)

Click on save and the cluster will be up and running in less than two minutes! 

Now it's time to do some provisioning. Remember we have selected Docker to be provisioned in a previous step, so we won't have to install it. But as we're going to have all needed containers ( nifi, elasticsearch, kibana, and so on ) orchestrated through a docker-compose.yml file ( always a neat and clear solution other than CLI ) let's install docker-compose to the master node ( all the magic happens here).

Click on SSH and then an interactive shell will pop up on your browser.

![Cloud Components](/images/08-Dataproc.png)

### Provision the master node with Docker Compose
  
- Download the Docker Compose binary into the /usr/local/bin directory with wget or curl :
    
        sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" \
    	-o /usr/local/bin/docker-compose
  
- Use chmod to make the Compose binary executable:
  
      	sudo chmod +x /usr/local/bin/docker-compose
  
- To verify the installation, use the following command which prints the Compose version:
  
      	docker-compose --version
    
- The output will look something like this:  
    
![Docker-Compose](/images/09_docker_compose.png)	

### Create docker-compose.yml file

This is a crucial step with all the settings. Check out that all needed iamges are present there: nifi,kafka,zookeeper,elastichsearch and kibana, environment variables, spinning up ordering, volumes, exposed ports,...

[docker-compose.yml](/scripts/docker-compose.yml) 


### Create a Docker network

Let's create a network named "nifinet" which have been set within docker-compose.yml file.

	docker network create nifinet --driver bridge	

### Spin up all containers

        docker-compose up -d 

The first time we issue this command will download nifi, zookeeper, kafka, elasticsearch an kibana images from Docker Hub, as we don't have it yet. You'll need to have signed up to a Docker account in order to use this service.

![docker-compose up -d  exit](/images/00_docker-compose-up.png)

Verify all 5 containers are up and none of them exited:

    docker ps -a

![docker-compose up -d  exit](/images/10_docker-compose-up.png)


   - **Nifi container** spins up the apache/nifi:latest image, which allows us to use a powerful and reliable system to process and distribute data, the  [Apache Nifi project](http://http://nifi.apache.org/download.html). It will be used as a single point of data entrace into the system, allowing us to ingest and transform data in a visual way. After that, data will be sinked to different locations such as hdfs, kafka brokers and mysql database.
   - **Zookeeper container** spins up the  confluentinc/cp-zookeeper:5.5.0 image. ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. We'll use it for managing kafka brokers.

   - **Kafka broker container** spins up the confluentinc/cp-kafka:5.5.0 image. Apache Kafka is a community distributed event streaming platform capable of handling trillions of events a day. Initially conceived as a messaging queue, Kafka is based on an abstraction of a distributed commit log. We'll use the image provided by [Confluence](https://www.confluent.io/what-is-apache-kafka/)

   - **Elasticsearch container** spins up the docker.elastic.co/elasticsearch/elasticsearch:7.6.2 image. Elasticsearch is a distributed, RESTful search and analytics engine capable of addressing a growing number of use cases. As the heart of the Elastic Stack, it centrally stores your data for lightning fast search, fine‑tuned relevancy, and powerful analytics that scale with ease.
   
   - **Kibana container** spins up the docker.elastic.co/kibana/kibana:7.6.2 image. Kibana is an free and open frontend application that sits on top of the Elastic Stack, providing search and data visualization capabilities for data indexed in Elasticsearch.
      
So, as you can see, we'll build up a bigdata ecosystem capable not only of ingesting all kinds of data but to extrac meaning in a visual way.

### Create a SSH tunnel to the VM instance (master node).

Through docker-compose.yml we have exposed port 8080 (from nifi container) to 8060 in localhost ( our vm instance which acts as master node). So we have to tunnel from our local machine ( laptop ) to the VM port 8060 to get access to Nifi Web UI. One way of doing it is by manually using PuTTY ,generating ssh-keys and installing them in the VM, connecting to the VM and specifying tunnel port in PuTTY. One easier way is through Google Cloud Standard Development Kit keeping in mind that you have to have Web browser Chrome already installed.

Windows as I have, I downloaded [Google Cloud Client](https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe)  and installed it.
Once installed their features are embedded within cmd or powershell commands and you can use gcloud extensions as if they belong to **Windows cmd** itself.

In Google Cloud there is a cheat sheet for Windows, Mac and Linux Systems.

![SSH tunnel](/images/00_gcloud.png)


- Open a Windows cmd console (choose your own case situation) and open a ssh tunnel through port 1080

     	gcloud compute ssh cluster-uoc-m ^
       	 --project=data-lakes-313014 ^
       	 --zone=europe-west6-b -- -D 1080 -N

This command will open a ssh tunnel through PuTTY generating ssh keys for you. That's why PuTTY UI opens and waits for you to add VMs' public key to PuTTY's cache and carry on connecting. 

![SSH tunnel](/images/10_gcloud.png)

- Launch chrome through ssh tunnel to access desired port. For this demo I've choosen Nifi UI which has port 8060 listening.

		C:\%ProgramFiles(x86)%\Google\Chrome\Application\chrome.exe ^
  		--proxy-server="socks5://localhost:1080" ^
  		--user-data-dir="%Temp%\cluster-uoc-m" http://cluster-uoc-m:8060

If you don't have the environment variable %ProgramFiles(x86)% set up to the actual path you can navigate to where chrome is an execute just the bit from chrome.exe onwards.

Then, a chrome window will pop up with the call made before (http://cluster-uoc-m:8060) and a nifi welcome page will show up redirecting to the main page after a while. 

Nifi is ready to go!


![Apache nifi](/images/00_nifi.png)

## Troubleshooting Common Issues with Kafka

First of all, let's have a look to the kafka arquitecture already set up through [docker-compose.yml](/scripts/docker-compose.yml)

Below is the extract related to kafka broker:

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


Check out that there are **two advertised listerners** on broker:9092 and localhost:19092. The whole thing of kafka connection (in our architecture) boils down to these two listeners.

Why two instead of the default one? 

The answer is that in order to establish a successful connection to kafka **two connections** must succeed:

1. The initial connection to a kafka broker (the bootstrap), which returns metadata to the client, including a list of all the brokers in the cluster and their connection endpoints.
2. The final connection, when the client connects to one (or more) of the brokers returned in the first step as required. If the broker has not been configured correctly, the connections will fail.

What usually happens is that you often only care for the first connection. Once it is succesful you say Aha! I'm connected! But you're not. 

If you expose port 9092 as you normally do, using the name of the broker and its port **(broker:9092)** as the advertised listener, the first connection against kafka (the one that gets the metadata) should succeed, but eventually, the second one should fail.

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

What happens is described in the graph below:

![Can't resolve host](/images/20_kafka_configuration.png)

The client (for example a Zeppelin notebook which is running on the master node of the VM ) cannot resolve  the advertised listener "broker:9092" as it has no means to resolve who "broker" is. If you should change the advertised listener to localhost:9092 should work, but any other Docker container trying to connect should fail, as to Docker containers point of view, kafka container remains in a different network an localhost would resolve to de Docker container itself.

![Can't resolve host](/images/listeners_kafka_localhost.png)

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

- Python test connection program [python_kafka_test_client.py](https://github.com/ajimeneo/dataproc-bigdata/blob/main/scripts/python_kafka_test_client.py). Use nano or other suitable text editor to create the python_kafka_test_client.py file using the available source.


- Create a [Dockerfile](https://github.com/ajimeneo/dataproc-bigdata/blob/main/scripts/Dockerfile). We will use this Dockerfile to create an image and spin up a container from it.

		docker build -t python_kafka_test_client .
	
- List images to check out that python_kafka_test_client image has been created

		docker images 

 ![Images list](/images/10_docker_build.png)
 
- Spin up a Docker container with python_kafka_test_client image

		docker run --network=nifinet --rm --name python_kafka_test_client \
        	--tty python_kafka_test_client broker:9092

The program expects a host as a first param. We supply "broker:9092" as the container spun up is within the same network.

 ![Success](/images/00_connection_successful.png)

- Once tested the connection, whe can list the topics using an ephemereus Docker container

		 docker run -it --rm --network nifinet \
		--name testKafkaTopicsList confluentinc/cp-kafka:5.5.0 \
		kafka-topics --list --bootstrap-server broker:9092

 ![Launching Test local to 9092](/images/10_launching_test_9092.png)
 
 Once some messages are produced to kafka broker ( via nifi, or kafka-console-producer for example ) we can check these messages out from topic mediciones:
 
 		docker run -it --rm --network nifinet \
		--name testKafkaTopicsList confluentinc/cp-kafka:5.5.0 \
		kafka-console-consumer --topic mediciones --from-beginning --bootstrap-server broker:9092

 ![Launching Test local to 19092](/images/20_launching_test_9092.png)
 
- Install confluent_kafka python package

		pip install confluent-kafka

 ![From localhost python to kafka](/images/10_install_confluent_kafka.png)
 
- Launch the python python_kafka_test_client.py program from the VM shell to connect to kafka broker
	
	 	python python_kafka_test_client.py localhost:19092
		
 ![Launching Test local to 19092](/images/10_launching_test_19092.png)
 
## Setting up Zeppelin

Zeppelin is provided to us from "WEB INTERFACES" option in Dataproc. Once clicked Zeppelin, Zeppelin welcome page will pop up.

It's crucial setting up Zeppelin interpreter for things to work. Otherwise we'll get endless error messages about not finding a specific class or interpreter.
We are setting two interpreters: spark and hive. Spark is given by default and we only have to set up a couple of properties. On the other hand, Hive interpreter is not provided so we'll create one.

### Spark Interpreter

Once we choose the option of the interpreter
 ![Spark Interpreter](/images/00_interpreter.png)
 
 and search for Spark
 
 ![Spark Interpreter](/images/10_interpreter.png)
 
 After scrolling down, there is an option to include some maven artifacts. This is the way that Zeppelin allows us to include any features that are not present by default.
 In our scenario, as we want kafka integration, we should include
 
	 org.apache.spark:spark-streaming-kafka-0-10_2.12:3.1.1
 
 ![Spark Interpreter](/images/20_interpreter.png)
 And save it.
 
 You may look it up in [Spark-kafka integration](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)
 
 ![Spark Interpreter](/images/30_interpreter.png)

It's always good politics to restart the interpreter any time you make a change or when things got weird about the interpreter and you cannot come up with a solution. Before you get nuts, try and restart the interpreter. One more thing, the bluish color of the interpreter points out that is enable. If you want to disable it, click it and it´ll became disable, showing it up with a greeish color.





