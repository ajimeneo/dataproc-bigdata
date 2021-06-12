# The Google Cloud Dataproc Bigdata Ecosystem Cookbook

How to build up from scratch a Bigdata Ecosystem using Dataproc Clusters supplied by Google Cloud Platform (GCP). Through Docker and Zeppelin we will build up a system with nifi, kafka, elasticsearch,kibana and zeppelin notebooks.

We will use [Santander Smart City](http://datos.santander.es/dataset/?id=datos-trafico) data to ingest into our system and play around. We'll choose to retrieve data under [JSON](http://datos.santander.es/api/rest/datasets/mediciones.json?items=482). Every minute, a JSON document with 482 individual traffic measures will be available to be consumed by an http GET/POST call through its API REST.

Some of the contents related to kafka ( specially connectivity with conteinerized-kafka are taken from [Why Can´t I Connect to Kafka](https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/) written by Robin Moffat). I believe his article is a remarkable explanation about how to connect to kafka and cannot be further improved. I've borrowed from this site, his own code and diagrams in order to explain how to make a connection to kafka and what could possibly go wrong.


Contents
============

- [What is this Cookbook](README.md#what-is-this-cookbook)
- [What is Google Cloud Dataproc](README.md#what-is-google-cloud-dataproc)
- [Building up the Dataproc Cluster](sections/01-BuildingUp.md#building-up-the-architecture-the-dataproc-cluster)
  - [Setting up the cluster](sections/01-BuildingUp.md#setting-up-the-cluster)
  - [Troubleshooting Common Issues with Kafka](sections/01-BuildingUp.md#Troubleshooting-common-issues-with-kafka)
  - [Setting Up Zeppelin](sections/01-BuildingUp.md#setting-up-Zeppelin)
- [Putting all agents working together](sections/02-PuttingAllTogether.md#putting-all-agents-working-together)
  - [Creating a flow with nifi](sections/02-PuttingAllTogether.md#nifi)
  - [Create a Map Visualization using Kibana](sections/02-PuttingAllTogether.md#create-a-map-visualization-using-kibana)
  - [Issue alerts with Zeppelin](sections/02-PuttingAllTogether.md#Issue-alerts-with-zeppelin)
 
## What is this Cookbook

This book is intended to be a proof of concept to spin up a fully bigdata ecosystem cluster from scratch using GCP.

We will suck data from online real data from the city of Santander, available to us through the Smart City web page, and ingest it into our system, which will be a Google Cloud Dataproc cluster, provisioned with nifi, kafka and elasticsearch Docker containers. The visual presentation will be done using kibana.

At the same time we will be able to interact with the system in real time with zeppelin notebooks.

In this cookbook we will cover all Data Engeneering platform key areas (Connect, Buffer, Processing Framework, Store, Visualize)

The proposed architecture flow will be as follows

![Arquitecura](/images/00-santander.png)


We will split the cookbook into two main parts:

**First**: Building up the architecture

**Second**: Putting all main agents ( nifi, elasticsearch, kibana, zeppelin) working together to ingest and extract meaning from the traffic data of the city of Santander

## What is Google Cloud Dataproc 

The **Google Cloud Dataproc** is a fully managed and highly scalable service for running Apache Spark, Apache Flink, Presto, and 30+ open source tools and frameworks for for batch processing, querying, streaming, and machine learning available through Google Cloud Platform (GCP). 

You may use Dataproc for data lake modernization, ETL, and secure data science, at planet scale. 

Why using GPC? Because you can build a BigData cluster right now without paying any money. I Signed up to its [free trial](https://cloud.google.com/free/docs/gcp-free-tier/#free-trial) at use its **300$** free credit for 90 days in order to do that. 

Thorugh Google Cloud Dataproc enhanced with Docker and Zeppelin notebooks we will be able to spin up a fully functional cluster in no time.

### Acknowledments

I want to express my deepest gratitude to my Data Lakes professor **Iraitz Montalbán** who has suffered me through this journey and has promptly and patiently answered my endless questions about architecture, configuration and all kinds of different issues. This cookbook wouldn't have been possible without his help and support.



