
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

All the data we're playing around is from a RestFul service http://datos.santander.es/api/rest/datasets/mediciones.json?items=482.

This URL is serving, every 60 seconds a traffic measure of every sensor. As items suggests, we will get 482 measurements each call.

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

### Create a Process Group

Drag and drop a Process Group into the canvas and type a name for it. I've choosen Santander traffic. 

![Process Group](/images/10_nifi.png)

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

So far we have succeed in ingesting some traffic data down the pipeline and learnt how to display the data ingested. Let's see how the json is splitted. The process is always the same. Start and stop the processor. Refresh from the canvas and check the messages that have been queued up. To do that keep in mind that you always have to be one step ahead ( you need a processor to sink the data in). As we want to rearrange the json a little bit ( modify names and dispose of some fields we won't be needing) then we'll use processor **JoltTransformJSON**. The data queued before entering JoltTransformation is the split relationship. As you can see we have now 482 messages which is the expected output. We have split 1 json with an array of 482 resources in 482 json messages

![Process Group](/images/350-nifi.png)	

Right click and "List queue".

![Process Group](/images/360-nifi.png)	

And there we have our json splitted

![Process Group](/images/370-nifi.png)	

Next step is tidying up the json a little bit, getting rid of some fields we don't need ( uri and dc:identifier ) and changing names from Spanish to English to make it more readable. In order to do that we'll use Jolt transformations. Everything you need about it is on this web page ![Jolt Transformation](https://jolt-demo.appspot.com/#inception)




| Podcast Episode: #050 Data Engineer, Scientist or Analyst - Which One Is For You?
|-----------------------------------------------------------------------------------
| In this podcast we talk about the diﬀerences between data scientists, analysts and engineers. Which are the three main data science jobs. All three are super important. This makes it easy to decide
| [Watch on YouTube](https://youtu.be/64TYZETOEdQ) \ [Listen on Anchor](https://anchor.fm/andreaskayy/episodes/050-Data-Engineer-Scientist-or-Analyst-Which-One-Is-For-You-e45ibl)

My YouTube video how to write to Kafka: <https://youtu.be/RboQBZvZCh0>

Acknowledments
https://linuxize.com/post/how-to-install-and-use-docker-compose-on-debian-10/
