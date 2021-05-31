# The Google Cloud Dataproc Bigdata Ecosystem Cookbook

How to build up from scratch a Bigdata Ecosystem using Dataproc Clusters supplied by Google Cloud Platform (GCP). Through Docker and Zeppelin we will build up a system with nifi, kafka, elasticsearch,kibana and zeppelin notebooks.

We will use Santander Smart City data to ingest into our system and play around

Some of the contents related to kafka ( specially connectivity with conteinerized-kafka are taken from this web page [Why Can´t I Connect to Kafka](https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/) written by Robin Moffat). I believe his article is a remarkable explanation about how to connect to kafka and cannot be further improved so I borrowed his own code and diagrams in order to explain how to make a connection to kafka and what could possibly go wrong.


Contents
============

- [What is this Cookbook](README.md#what-is-this-cookbook)
- [Building up the Dataproc Cluster](sections/01-BuildingUp.md#building-up-the-architecture-the-dataproc-cluster)
  - [What is Google Cloud Dataproc](sections/01-BuildingUp.md#what-is-google-cloud-dataproc)
  - [First Steps](sections/01-BuildingUp.md#first-steps)
  - [Connecting to Apache Kafka](sections/01-BuildingUp.md#Connecting-to-kafka)
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


# Putting all agents working together

## Using Apache Nifi


















		
| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/

