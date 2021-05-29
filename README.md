# dataproc-nifi-kafka

![Cloud Components](/images/00-Dataproc.png)

### Data Engineer

Introduction
============

## Contents

- [What is this Cookbook](README.md#what-is-this-cookbook)
- [Data Engineer vs Data Scientist](README.md#data-engineer-vs-data-scientist)
  - [Data Engineer](01-Introduction.md#data-engineer)
  - [Data Scientist](01-Introduction.md#data-scientist)
  - [Machine Learning Workflow](01-Introduction.md#machine-learning-workflow)
  - [Machine Learning Model and Data](01-Introduction.md#machine-learning-model-and-data)
- [My Data Science Platform Blueprint](01-Introduction.md#my-data-science-platform-blueprint)
  - [Connect](01-Introduction.md#connect)
  - [Buffer](01-Introduction.md#buffer)
  - [Processing Framework](01-Introduction.md#processing-framework)
  - [Store](01-Introduction.md#store)
  - [Visualize](01-Introduction.md#visualize)
- [Who Companies Need](01-Introduction.md#who-companies-need)
- [Why Big Data](03-AdvancedSkills.md#why-big-data)
    - [Planning is Everything](03-AdvancedSkills.md#planning-is-everything)
    - [The Problem with ETL](03-AdvancedSkills.md#the-problem-with-etl)
    - [Scaling Up](03-AdvancedSkills.md#scaling-up)
    - [Scaling Out](03-AdvancedSkills.md#scaling-out)
    - [When not to Do Big Data](03-AdvancedSkills.md#please-dont-go-big-data)

## What is this Cookbook

I get asked a lot:
"What do you actually need to learn to become an awesome data engineer?"

Well, look no further. you'll find it here!

If you are looking for AI algorithms and such data scientist things,
this book is not for you.

**How to use this Cookbook:**
This book is intended to be a starting point for you. It is not a training! I want to help you to identify the topics to look into and becoming an awesome data engineer in the process.

It hinges on my Data Science Platform Blueprint, check it out below. Once you understand it, you can find in the book tools that fit into each key area of a Data Science platform (Connect, Buffer, Processing Framework, Store, Visualize).

Select a few tools you are interested in, research and work with them.

Don't learn everything in this book! Focus.

**What types of content are in this book?**
You are going to find five types of content in this book: Articles
I wrote, links to my podcast episodes (video & audio), more than 200
links to helpful websites I like, data engineering interview questions
and case studies.

**This book is a work in progress!**
As you can see, this book is not finished. I'm constantly adding new
stuff and doing videos for the topics. But obviously, because I do this
as a hobby my time is limited. You can help making this book even
better.

**Help make this book awesome!**
If you have some cool links or topics for the cookbook, please become a
contributor on GitHub: <https://github.com/andkret/Cookbook>. Fork the

Data Engineer vs Data Scientist
-------------------------------

| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)



### Apache Kafka

#### Why a message queue tool?

#### Kafka architecture

#### What are topics

#### What does Zookeeper have to do with Kafka

#### How to produce and consume messages

My YouTube video how to set up Kafka at home:
<https://youtu.be/7F9tBwTUSeY>

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

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


