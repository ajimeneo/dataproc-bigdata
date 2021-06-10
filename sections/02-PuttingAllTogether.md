
# Putting all agents working together

  - [Flow creation with nifi](02-PuttingAllTogether.md#flow-creation-with-nifi)
  - [Create a Map Visualization using Kibana](02-PuttingAllTogether.md#create-a-map-visualization-using-kibana)
  - [Issue alerts and other useful queries with Zeppelin](02-PuttingAllTogether.md#zeppelin)


Once we have set up the datproc cluster and all containers are up and running ( nifi, zookeeper, kafka, elasticsearch and kibana) it's time to have a hands-on approach.

This is the architecture we have built up:

-- photo

## Flow creation with nifi

Nifi UI web page is listening on port 8060 and we have to access it through a SSH tunnel. As we have explained on the previous section [Setting up the cluster](01-BuildingUp.md#Create-a-SSH-tunnel-to-the-VM-instance-(master-node)), let's dive into nifi.

Nifi is very powerful ETL tool that allows us to ingest massive data from any endpoint in a visual fashion. Everything's designed from a process-flow perspective. Data ingested into the pipeline is convert into Flowfiles which are the pieces of data we are manipulating. Processors define actions over these Flowfiles.

So processors are the pieces we are going to determine once we drag them into the canvas.

Let's jump into it!

One more thing before we go.

All the data we're playing around is from a Restful service http://datos.santander.es/api/rest/datasets/mediciones.json?items=482.

This URL serves, each sensor's traffic measurement, every 60 seconds. As items suggests, we will get 482 measurements each call.

This is a extract from the whole call. ![measurements.json](/files/measurements.json)
		
	{"summary":{"items":482,"items_per_page":50,"pages":10,"current_page":1},
	 "resources":[{
		      "ayto:ocupacion":"2",
		      "ayto:medida":"1001",
		      "ayto:idSensor":"1001",
		      "ayto:intensidad":"240",
		      "dc:modified":"2021-06-06T11:08:00Z",
		      "dc:identifier":"1001-e311e9c0-c6b7-11eb-9137-005056a43242",
		      "ayto:carga":"10",
		      "uri":"http://datos.santander.es/api/datos/mediciones/1001-e311e9c0-c6b7-11eb-9137-005056a43242.json"},
	 	      {
		      "ayto:ocupacion":"0",
		      "ayto:medida":"1002",
		      "ayto:idSensor":"1002",
		      "ayto:intensidad":"0",
		      "dc:modified":"2021-06-06T11:08:00Z",
		      "dc:identifier":"1002-e312c8e8-c6b7-11eb-9137-005056a43242",
		      "ayto:carga":"0",
		      "uri":"http://datos.santander.es/api/datos/mediciones/1002-e312c8e8-c6b7-11eb-9137-005056a43242.json"},
		      ...
		      ]
        }


So keep in mind that we have to transform this json into 482 individual json.

# Overview of the whole process

Below is the image of the whole process. We're going to build it step by step. It consists of several processes, from ingesting traffic data from a exposed URL to splitting, jolt tranforming, enriching and delivering the JSON message obtained into 3 different sinks: 
- A **kafka** sink, where some spark process, from a Zeppelin notebook,  will consume its data to produce some alerts
- A **hdfs** sink, where some hive process, from a Zeppelin notebook, will dig in into the historical data to find some valueable insight
- A **elasticsearch** sink, which will be used for presenting a dashboard with real-time traffic status. 

![Process Group](/images/530-nifi.png)

### Create a Process Group

Drag and drop a Process Group into the canvas and type a name for it. I've choosen Santander traffic. 

![Process Group](/images/10_nifi.png)

All the processors we're going to create will exist within this group processor. Double click it. Then, every processor we may add will fall into this group.

## **InvokeHTTP** (Ingesting data into the pipeline from a URL)

Drag and drop a processor into the canvas.

![Process Group](/images/240-nifi.png)

Search for **InvokeHTTP**

![Process Group](/images/20_nifi.png)

Click on Add. After that a box with a label InvokeHTTP appears on it. Right click on the mouse and hit Configure from the list.

![Process Group](/images/40-nifi.png)

This processor allows us to ingest data into the pipeline from a supplied URL. So, fill in with http://datos.santander.es/api/rest/datasets/mediciones.json?items=482

![Process Group](/images/50-nifi.png)

Let's schedule the call to be issued every 60 seconds. Then, every 60 seconds we will get fresh data of how the traffic is in the city of Santander.
	
![Process Group](/images/60-nifi.png)	

As nifi is concerned, the flow is a pipeline with a beginning and a ending. So, every time we add a processor to the flow it is mandatory to set every the destination and origin (called relationship) for every possible flow.

That's why some warnings pops up if we haven't properly set the processor:

![Process Group](/images/30-nifi.png)	

**In nifi there is always an origin processor, a destination processor and a List queue in the middle**. The queue is the place where transformed messages are pile up.
	
In order to have a queue link **InvokeHTTP** to another processor. Which one will suit? Many times, we don't know the destination ( the next processor to use ) so to work it out, it's always good politics to have a look at some messages from the queue. Then it will come clear to us, the transformations that have been made and which ones will be needed. Use then **Wait** procesor. In order to have a queue link **InvokeHTTP** to **Wait** by dragging the arrow that appears at the center of the processor and drop it to another processor. Then a Queue appers between the two processors. 

### Checking the queue

But let's check that we have received data. Start the InvokeHTTP processor. Stop it. And from the canvas, right click and Refresh. Once a processor process the data, its transformed data will be added to the queue. Check that there's 1 message queued as expected. That's the flowfile with json format waiting for us.

Right click and "List queue".

![Process Group](/images/300-nifi.png)	

Then you can see all the available data waiting to be ingested into the pipeline. 

![Process Group](/images/310-nifi.png)

As you check the message clicking on the eye icon, you come to the conclusion that you need 482 indivual json, each one representing 1 measurement, and not only 1 json with and array of 482 measures. There's when SplitJson processor comes in handy.

![Process Group](/images/100-nifi.png)


## **SplitJson** (Split a JSON document into several)

Get rid of Wait processor. We'll need **SplitJson** processor to do the task of transforming our individual flowfile into 482 flowfiles. Drag and drop a processor into the canvas. Search for **SplitJson** and click on Add. Link it with **InvokeHTTP**.

![Process Group](/images/290-nifi.png)

We're going to handle only "Response" flowfile because if everything's fine we'll get the json flowfile through this relationship. We terminate all the other relationships.	

![Process Group](/images/250-nifi.png)

Set JsonPathExpression property from the tab properties of the **SplitJson** to $.resources. This way the original json will split into individual json using the field provided (**resources**)
	
![Process Group](/images/260-nifi.png)	

Terminate the relationships you won't be needing. We'll terminate failure relationship. We'll use original to redirect the original json to another flow, and the split relationship down the pipeline.

![Process Group](/images/270-nifi.png)	

Right click and "List queue". 

![Process Group](/images/360-nifi.png)

Then you can see all the available data waiting to be ingested into the pipeline. There has to be exactly 482 messages as we splitted one json into its 482 messages. 

![Process Group](/images/370-nifi.png)	


## **JoltTransformJSON** (Modify a JSON document)

So far we have succeed in ingesting some traffic data down the pipeline and learnt about how to display the data ingested. It comes clear that we want get rid of some fields we won't be needing ( uri and dc:identifier ) and rename some fields from Spanish to English to make it more readable. In order to do that we'll use Jolt transformations. Everything you need about it is on this web page <https://jolt-demo.appspot.com>.

We'll add then, **JoltTransformJSON** processor. The data queued before entering JoltTransformationJSON is the split relationship.

![Process Group](/images/350-nifi.png)	

To configure **JoltTransformJSON** processor go to properties tab and click ADVANCED bottom. Use the [Jolt specification provided](/files/JoltTransformationJSON.json), insert a JSON input (take one of the json messages) and click on transform. This way you will be able to see the output. Once you get the correct one, click on validate. Every time a json enters this transformation, the jolt transformation is applied and a new JSON will come out.

This is the expected Jolt transformation output:

![Process Group](/images/380-nifi.png)	

Let's see how the json is jolted. The process is always the same. Start and stop the processor. Refresh from the canvas and check 482 messages that have been queued up.

![Process Group](/images/390-nifi.png)

## **LookupRecord** (Look up record)

Now it's time to enrich the flow with data from another source. In our case from a csv document, which holds the location (latitude and longitude) and identification of each sensor. His id is the same idSensor that holds the ingested data.

So our goal is to end up having the same original JSON enriched with his latitude and longitude. To join these two datasets we will need a common field between the two:
- "idSensor" in our json data 
- "sensor" in the csv file. 
 
In the end we will use this location to pinpoint the exact location of sensors and their measurements on a kibana map. Find [sensors.csv](https://raw.githubusercontent.com/IraitzM/Santander/master/location.csv) file on an open repository in GitHub. 

First of all we have to put the sensors.csv file into our own file system. 
Second, we have to set LookupRecord processor configuration

### Lookup file Sensors.csv flow

Up to this point we need to set another flow that takes a file from GitHub and puts it on our own file system (nifi container file system).
We'll use a **InvokeHTTP** processor to get the file from a URL as we've done before, a **UpdateAttribute** processor to change some of the flowfile attribute's names from and make them more readable. It's always good practice to have some meaningful name as "sensors.dsv" than a UUID file name. Lastly, we'll add **PutFile** processor to put the flowfile into our own file system.

![Process Group](/images/480-nifi.png)

- **InvokeHTTP** properties

Set its properties

![Process Group](/images/490-nifi.png)

Check the queue. Its output is similar to

![Process Group](/images/500-nifi.png)

- **UpdateAttribute** properties

Press + button and add **filename** property. It has to be "filename" and not another, because filename is the flowfile built-in name for "the name of this flowfile". This way, we'll be able to rename the attribute flowfile, which has uuid by default, to one of our choosing.

![Process Group](/images/510-nifi.png)

- **PutFile** properties

The final destination of our file will be our own file system. Rebember that nifi is running inside a Docker container, and is image is based on Linux, so we have to choose a destination which nifi has rights access. That could be the entrypoint, which is "/opt/nifi/nifi-current". We're adding the bit /files which points out a file that is not existing, but it is created on the fly.

![Process Group](/images/490-nifi.png)

### LookupRecord processor configuration

![Process Group](/images/400-nifi.png)

Its configuration is a little bit tricky. You have to provide a **RecordReader**, **RecordWriter** and **LookupService** mainly. 

![Process Group](/images/540-nifi.png)

To create a Controller Service:

![Process Group](/images/420-nifi.png)

- Record Reader. Create a Controller Service that reads from a JSON, our main flow. The one provided by nifi is ok.

![Process Group](/images/410-nifi.png)

- Record Writer. Create a Controller Service that writes a JSON record as we want to enrich our json main flow. The one provided by nifi is ok. Take the same steps and choose JsonRecordSetWriter

![Process Group](/images/440-nifi.png)

- Lookup Service. The file to enrich our flow is a csv file. Then, create a Controller Service that looks up in a CSV file. Select CSVRecordLookupService. Then set the Properties of the controller.

	- The CSV file path to look up to, will be:  **/opt/nifi/nifi-current/sensores.csv**. Remember that this is a path that has to exist in our docker nifi container.
	- The lookup key column will be "sensor" as is the csv field that will be joined and searched for.

![Process Group](/images/460-nifi.png)

The entire data found on the csv file, formatted in a JSON way, will be returned to the calling flowfile to a json field of our choosing: In our case "/location"
We must create a property, "key" to point out the field to join to the csv field (sensor). That'll be "/idSensor"

![Process Group](/images/540-nifi.png)


Add a wait processor and check that everything run smoothly.

![Process Group](/images/550-nifi.png)

Checking the queue. There's our json enriched with location.

![Process Group](/images/570-nifi.png)


It's time to set the final destination of our messages.

## Flow destinations

As we have said before, we have 3 main destinations for our data: a kafka broker, hdfs and elasticsearch engine

![Process Group](/images/560-nifi.png)

- **Kakfa destination**

![Process Group](/images/580-nifi.png)

 As we've mentioned on setting the datproc cluster, kafka broker is listening on port 9092 as a docker container point of view. Then the advertised listerner should be broker:9092. Fill in the topic name with one of your choosing, for example, "mediciones". Create it before producing some messages to this topic. 
 
 You can do that, spinning up a ephemeral container like this:
	
	   docker run -it --rm --network nifinet \
  		--name testKafkaTopicsList confluentinc/cp-kafka:5.5.0 \
  		kafka-topics --create --topic mediciones --partitions 1 replication-factor 1 --bootstrap-server broker:9092


Properties should be set like the image below.

![Process Group](/images/610-nifi.png)

- **Elasticsearch destination**

![Process Group](/images/590-nifi.png)

Let's set the properties. Keep in mind that elasticsearch's IP is the elasticsearch container's IP that you can find out issuing

		docker newtwork inspect nifinet

![Process Group](/images/20_nifinet.png)


Check the container named elasticsearch and look up its IP. In our example is 172.19.0.5.
Then set elasticsearch's url to http://172.19.0.5:9200

Set the other properties this way. One particular thing to point out is index propery. As we're inserting data into elasticsearch engine, we'll do it as an index. Just name the index. It'll be mediciones as well ( like the topic, buy you could've chosen whatever name you feel like ).

![Process Group](/images/620-nifi.png)
![Process Group](/images/630-nifi.png)

- **HDFS destination**

![Process Group](/images/600-nifi.png)

Set the properties. We are in need of these hadoop config files **hdfs-site.xml** and **core-site.xml**.

They can be found in our nifi container at:

	/hadoop-conf/hdfs-site.xml
	/hadoop-conf/core-site.xml

To check that spin up a docker container.

    docker exec -it --rm nifi /bin/bash && cd /hadoop-conf

As we need them on client side ( it's where it stands hadoop) add the path as a volume as we've done on our docker-compose.yml file. The bit related to this:

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
      - broker
	
This way everything on /hadoop-conf path (our two files) will be at /etc/hadoop/conf path, which is what we wanted. The destination path is set to /tmp, which we know it exists on client side and to which we have access rights.

![Process Group](/images/640-nifi.png)

So down the pipeline, we can find all ingested messages on our hdfs file system, particularly on /tmp:

	hdfs dfs -ls /tmp
	


## Create a Map Visualization using Kibana

### What is kibana?

[Kibana](https://www.elastic.co/what-is/kibana) is an free and open frontend application that sits on top of the Elastic Stack, providing search and data visualization capabilities for data indexed in Elasticsearch. Kibana also acts as the user interface for monitoring, managing, and securing an Elastic Stack cluster.  

That lead us to the point of what the devil is Elasticsearch? To put it shortly, is a query engine and as its welcome page says, Elasticserach is part of the ELK Stack and is built on Lucene, the search library from Apache, and exposes Lucene’s query syntax. It’s such an integral part of Elasticsearch that when you query the root of an Elasticsearch cluster, it will tell you the Lucene version. We have elasticsearch container exposed on 9200. If we open elasticsearch on a web browser typing what's next on a cmd console:

	C:\>%ProgramFiles(x86)%\Google\Chrome\Application\chrome.exe ^
		  --proxy-server="socks5://localhost:1080" ^
		  --user-data-dir="%Temp%\cluster-uoc-m" http://cluster-uoc-m:9200

		
![Kibana](/images/10-elasticsearch.png)

There it is the Lucene version!

## Mapping the ingested data into Elasticsearch

So, first things first. In order to create a visualization in kibana we must insert some data into elasticsearch. Rebember we'ver already done that in nifi, through **PutElasticsearchHttpRecord** processor. And all that data is delivered to index "mediciones"

Kibana UI rests on its own Docker container exposed on port 5601

	C:\>%ProgramFiles(x86)%\Google\Chrome\Application\chrome.exe ^
		  --proxy-server="socks5://localhost:1080" ^
		  --user-data-dir="%Temp%\cluster-uoc-m" http://cluster-uoc-m:5601

![Kibana](/images/10-kibana.png)

Then kibana welcome page opens:

![Kibana](/images/00-kibana.png)

We'll use elasticsearch query language (https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) to find out what's happening to our ingested data. There are many useful sites where [queries syntax] (!https://logz.io/blog/elasticsearch-queries) are explained in detail, so help yourself and found one to your liking.


Go to console and let's make out first query

	GET /_cat/indices
	
As you can see, the index mediciones is there. 

![Kibana](/images/20-elasticsearch.png)

You can find this index other than programatically going through Management option

![Kibana](/images/30-elasticsearch.png)

![Kibana](/images/60-elasticsearch.png)


But what data does it hold?

Go to Discover and select the appropiate interval. it's default it's last 15 minutes to it is very likely that you won't find any data even though it is there. So broad the interval to make sure it is wide enough to discover your data.

![Kibana](/images/30-elasticsearch.png)

Now there is some data, and if you expand a document you can see all of our ingested json's:

![Kibana](/images/30-elasticsearch.png)


There's one important thing to keep in mind. Elasticsearch, by default, infers the schema of the data that is ingesting and creates an automate mapping to do that. As we are ingesting a json, all of the fields will be treated as strings. Go to Management > Elastisearch > Index Management


![Kibana](/images/50-elasticsearch.png)

There's mediciones index. Click on it. 

![Kibana](/images/60-elasticsearch.png)

Go to Mapping tab. There's the mapping that has been applied to our data in JSON format.

![Kibana](/images/80-elasticsearch.png)

And check that all fields have been retrieved as "text" type. 

![Kibana](/images/90-elasticsearch.png)

That's not really what we wanted because in order to make a map visualization, that is, to allow kibana to evaluate lat and long fields and treat them as geolocaltion points drawable on a map, it is mandatory those fields to be of type geo_point ( this is a "proprietary" type in kibana ).

So, how can we force, elasticsearch to translate them into such geo_points ? Templates come to our rescue.







| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)


Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/
