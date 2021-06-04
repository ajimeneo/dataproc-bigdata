# Building up the architecture: The Dataproc Cluster

## Contents

  - [What is Google Cloud Dataproc](01-BuildingUp.md#what-is-google-cloud-dataproc)
  - [First Steps](01-1-FirstSteps.md#first-steps)
  - [Connecting to Apache Kafka](01-BuildingUp.md#Connecting-to-kafka)
  - [Setting Up Zeppelin](01-BuildingUp.md#setting-up-Zeppelin)


## What is Google Cloud Dataproc 

The **Google Cloud Dataproc** is a fully managed and highly scalable service for running Apache Spark, Apache Flink, Presto, and 30+ open source tools and frameworks for for batch processing, querying, streaming, and machine learning available through Google Cloud Platform (GCP). 

You may use Dataproc for data lake modernization, ETL, and secure data science, at planet scale. 

Why using GPC? Because you can build a BigData cluster right now without paying any money. I Signed up to its [free trial](https://cloud.google.com/free/docs/gcp-free-tier/#free-trial) at use its **300$** free credit for 90 days in order to do that. 

Thorugh Google Cloud Dataproc enhanced with Docker and Zeppelin notebooks we will be able to spin up a fully functional cluster in no time.

## First Steps

Once you set up the account, create a new Project, and search de Dataproc section.

![Cloud Components](/images/00-Dataproc.png)

Then hit Create a Cluster and a window will pop up. Fill in the blanks with an appropriate cluster name. Choose a region close to your location ( in my case [europe-west6](https://cloud.google.com/compute/docs/regions-zones)) and a cluster type. If you want to save as much credit as you can, choose single node, buy if you want to recreate a real cluster then choose a standard one. Image type and version should be ok with the default ( 2.0-debian10 ), enable component gateaway to allows us access to the web Interfaces and select Docker and Zeppelin as Optional components.







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

What usually happens is that you often only care for the first connection. Once it is succesful you say Aha! I'm connected! But you're not. 

If you expose port 9092 as you normally do, and uses ad advertised listener the name of the broker and its port (broker:9092) then the first connection against kafka (the one that gets the metadata) should succeed, but the second one should fail.

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

The client (for example a Zeppelin notebook which is running on the master node of the VM ) cannot resolve  the advertised listener "broker:9092" as it has no means to resolve who "broker" is.

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





