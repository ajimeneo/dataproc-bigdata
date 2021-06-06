
# Putting all agents working together

  - [Creating a flow with nifi](02-PuttingAllTogether.md#creating-a-flow-with-nifi)
  - [Create a Map Visualization using Kibana](sections/02-PuttingAllTogether.md#kibana)
  - [Issue alerts and other useful queries with Zeppelin](sections/02-PuttingAllTogether.md#zeppelin)


Once we have set up the datproc cluster and all containers are up and running ( nifi, zookeeper, kafka, elasticsearch and kibana) it's time to have a hands-on approach.

This is the architecture we have built up:

-- photo

## Creating a flow with nifi

Nifi web page is listening on port 8060 and we have to access it through a SSH tunnel. As we have explained on the previous section [Setting up the cluster](01-BuildingUp.md#Create-a-SSH-tunnel-to-the-VM-instance-(master-node)), let's dive into nifi.

Nifi is very powerful ETL tool that allows as to ingest massive data from any endpoint in a visual fashion. Everything is designed from a process-flow perspective. Data ingested into the pipeline is convert into Flowfiles which are the piece of data we are manipulating. Processors define actions over these Flowfiles.

So processors are the pieces we are going to define once we drag them into the canvas.

Let's jump into it!

One more thing before we go. 
All the data we're playing around is from a RestFul service https:// and the data is provided to us every 60 seconds in a JSON format. It's time to see this data by ourselves.

### Create a Processor Group

Drag and drop a Processor Group into the canvas and type a name for it. I've choosen Santander traffic. 

![Process Group](/images/10_nifi.png)

All the processors we're going to create will exist within this group processor. Double click it. Then, every processor we may add will fall into this group.

Drag and drop a processor into the canvas.

![Process Group](/images/240-nifi.png)

Search for **InvokeHTTP**

![Process Group](/images/20_nifi.png)

Click right bottom and hit Configure

![Process Group](/images/40-nifi.png)







		
| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/
