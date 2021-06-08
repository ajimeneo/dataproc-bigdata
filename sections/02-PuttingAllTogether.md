
# Putting all agents working together

  - [Flow creation with nifi](02-PuttingAllTogether.md#flow-creation-with-nifi)
  - [Create a Map Visualization using Kibana](sections/02-PuttingAllTogether.md#kibana)
  - [Issue alerts and other useful queries with Zeppelin](sections/02-PuttingAllTogether.md#zeppelin)


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

## The whole process

Below is the image of a nifi flow. We're going to build it step by step. It consists of several processes, from ingesting traffic data from a exposed URL to transforming it, enriching it on its way to 3 sinks: a kafka sink, where a spark process will consume its data to produce some alerts using a Zeppelin notebook, a hdfs sink where we'll use hive to process this historic data and a elasticsearch sink, which will be used for presenting a dashboard with a real-time traffic status. 

![Process Group](/images/530-nifi.png)

### Create a Process Group

Drag and drop a Process Group into the canvas and type a name for it. I've choosen Santander traffic. 

![Process Group](/images/10_nifi.png)

### Create the flow using processors

All the processors we're going to create will exist within this group processor. Double click it. Then, every processor we may add will fall into this group.

Drag and drop a processor into the canvas.

![Process Group](/images/240-nifi.png)

Search for **InvokeHTTP**

![Process Group](/images/20_nifi.png)

Click right bottom and hit Configure

![Process Group](/images/40-nifi.png)

This processor allows us to ingest data into the pipeline from a supplied URL. So, fill in with http://datos.santander.es/api/rest/datasets/mediciones.json?items=482

![Process Group](/images/50-nifi.png)

Let's schedule the call to be issued every 60 seconds. Then, every 60 seconds we will get fresh data of how the traffic is in the city of Santander.
	
![Process Group](/images/60-nifi.png)	

As nifi is concerned, the flow is a pipeline with a beginning and a ending. So, every time we add a processor to the flow it is mandatory to set every the destination and origin (called relationship) for every possible flow.
That's why some warnings pops up if we haven't properly set the processor:

![Process Group](/images/30-nifi.png)	
	
We're going to handle only "Response" flowfile because if everything's fine we'll get the json flowfile through this relationship. We terminate all the other relationships.	

![Process Group](/images/250-nifi.png)

We'll need **SplitJson** processor to do the task of transforming our individual flowfile into 482 flowfiles.

Drag a **SplitJson** processor. 

![Process Group](/images/280-nifi.png)	

Link it with **InvokeHTTP**. In order to do that, drag the arrow that appears at the center of the processor and drop it to another processor. Then a Queue appers between the two processors. This is the queue where messages will pile up.

Set JsonPathExpression property from the tab properties of the **SplitJson** to $.resources. This way the original json will split into individual json using the field provided (**resources**)
	
![Process Group](/images/260-nifi.png)	

Terminate the relationships you won't be needing. We'll terminate failure relationship. We'll use original to redirect the original json to another flow, and the split relationship down the pipeline.

![Process Group](/images/270-nifi.png)	
	
But let's check that we have received data. Start the InvokeHTTP processor. Stop it. And from the canvas, right click and Refresh. Data will be added to the queue.
Check that 1 message is queued as expected. So it has to be flowfile with json format waiting for us

![Process Group](/images/290-nifi.png)	

Right click and "List queue".

![Process Group](/images/300-nifi.png)	

Then you can see all the available data waiting to be ingested into the pipeline. 

![Process Group](/images/310-nifi.png)	

Check that our json is there clicking view content. Press the eye icon.
	
![Process Group](/images/100-nifi.png)	

So far we have succeed in ingesting some traffic data down the pipeline and learnt how to display the data ingested. Let's see how the json is splitted. The process is always the same. Start and stop the processor. Refresh from the canvas and check the messages that have been queued up. To do that keep in mind that you always have to be one step ahead ( you need a processor to sink the data in). As we want to rearrange the json a little bit ( modify names and dispose of some fields we won't be needing) then we'll add **JoltTransformJSON** processor. The data queued before entering JoltTransformationJSON is the split relationship. As you can see we have now 482 messages which is the expected output. We have split 1 json with an array of 482 resources in 482 json messages

![Process Group](/images/350-nifi.png)	

Right click and "List queue".

![Process Group](/images/360-nifi.png)	

And there we have our json splitted

![Process Group](/images/370-nifi.png)	

Next step is tidying up the json a little bit, getting rid of some fields we don't need ( uri and dc:identifier ) and changing names from Spanish to English to make it more readable. In order to do that we'll use Jolt transformations. Everything you need about it is on this web page <https://jolt-demo.appspot.com>

To configure **JoltTransformJSON** processor go to properties tab and click ADVANCED bottom. Use the [Jolt specification provided](/files/JoltTransformationJSON.json), insert a JSON input (take one of the json messages) and click on transform. This way you will be able to see the output. Once you get the correct one, click on validate. Every time a json enters this transformation, the jolt transformation is applied and a new JSON will come out.

This is the expected Jolt transformation output:

![Process Group](/images/380-nifi.png)	

So Start the **JoltTransformJSON** processor, stop it and refresh everything from the canvas. Go the the queue after JoltTransformJSON tranformation and 
see 482 messages transformed:

![Process Group](/images/390-nifi.png)

Now it's time to enrich the flow with data from a csv document, which holds the sensors location (latitude and longitude). We will end up having the same original JSON but enriched with his latitude and longitude. To join these two datasets we will need a common field which will be "idSensor" in our json data, and "sensor" in the csv file.. In the end we will use this location to pin point the sensors in a kibana map. This [sensors.csv](https://raw.githubusercontent.com/IraitzM/Santander/master/location.csv) file is in an open repository in GitHub. We'll use **LookupRecord** processor.

![Process Group](/images/400-nifi.png)

Its configuration is a little bit tricky. You have to provide a RecordReader, RecordWriter and LookupService mainly. To creader a Controller Service:

![Process Group](/images/420-nifi.png)

- Record Reader. Create a Controller Service that reads from a JSON. The one provided by nifi is ok.

![Process Group](/images/410-nifi.png)

- Record Writer. Create a Controller Service that writes a JSON record. The one provided by nifi is ok. Take the same steps and choose JsonRecordSetWriter

![Process Group](/images/440-nifi.png)

- Looukup Service. Create a Controller Service that looks up in a CSV file. Choose CSVRecordLookupService. Then set the Properties of the controller.

The CSV file to look up to will be at this path **/opt/nifi/nifi-current/sensores.csv**. Remember that this is a path that has to exist in docker nifi container.
The lookup key column will be "sensor" as is the field to be joined to.

![Process Group](/images/460-nifi.png)


To get to this point we need to set another flow that takes a file from GitHub and puts it on our own file system (nifi container file system).
We'll use a **InvokeHTTP** processor to get the file from a URL as we've done before, a **UpdateAttribute** processor to change some of the flowfile attribute's names from and make them more readable. It's always good practice to have some meaningful name as "sensors.dsv" than a UUID file name. Lastly, we'll add **PutFile** processor to put the flowfile into our own file system.

![Process Group](/images/480-nifi.png)

- **InvokeHTTP** properties

Check the queue. Its output is similar to

![Process Group](/images/500-nifi.png)


![Process Group](/images/490-nifi.png)

- **UpdateAttribute** properties

Press + button and add **filename** property. It has to be "filename" and not another, because filename is the flowfile built-in name for "the name of this flowfile". This way, we'll be able to rename the attribute flowfile, which has uuid by default, to one of our choosing.

![Process Group](/images/510-nifi.png)

- **PutFile** properties

The final destination of our file will be our own file system. Rebember that nifi is running inside a Docker container, and is image is based on Linux, so we have to choose a destination which nifi has rights access. That could be the entrypoint, which is "/opt/nifi/nifi-current". We're adding the bit /files which points out a file that is not existing, but it is created on the fly.

![Process Group](/images/490-nifi.png)

| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/
