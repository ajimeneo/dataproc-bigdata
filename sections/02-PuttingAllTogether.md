
# Putting all agents working together

  - [Creating a flow with nifi](02-PuttingAllTogether.md#creating-a-flow-with-nifi)
  - [Create a Map Visualization using Kibana](sections/02-PuttingAllTogether.md#kibana)
  - [Issue alerts and other useful queries with Zeppelin](sections/02-PuttingAllTogether.md#zeppelin)


Once we have set up the datproc cluster and all containers are up and running ( nifi, zookeeper, kafka, elasticsearch and kibana) it's time to have a hands-on approach.

This is the architecture we have built up:

-- photo

## Creating a flow with nifi

Nifi web page is listening on port 8060 and we have to access it through a SSH tunnel. As we have explained on the previous section [Setting up the cluster](01-BuildingUp.md#Setting-up-the-cluster), let's dive into nifi.



















		
| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/
