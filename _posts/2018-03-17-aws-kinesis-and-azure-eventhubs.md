---
title: A closer look at data streaming capabilities of Amazon Kinesis and Azure Event Hubs
layout: post
tags: analytics, architecture
excerpt: Microsoft Azure and Amazon Web Services both offer capabilities in the area of management and analytics on streaming data. In this post I'm looking a bit closer at how Azure Event Hubs and Azure Stream Analytics stack up against AWS Kinesis Firehose, Kinesis Data Streams and Kinesis Data Analytics.
---
In an ever increasing pace of business, competition, data generation and digitalization, businesses are looking for improved ways of gaining insights and making decisions faster than before. For certain scenarios this means moving away from weekly reporting to near real-time insights based on what's happening just at this moment.

Microsoft Azure and Amazon Web Services both offer capabilities in the areas of ingestion, management and analysis of streaming event data. In this post I'm looking a bit closer at how Azure Event Hubs and Azure Stream Analytics stack up against AWS Kinesis Firehose, Kinesis Data Streams and Kinesis Data Analytics.

## Scenarios

### Data ingestion for batch processing
Both Amazon and Microsoft Azure offer services that'll help you ingest large amount of event data for furhter processing. Amazon has its separate Kinesis Firehose service which is designed to ingest and forward event data for persistence while Microsoft Azure has this baked in as a part of its Event Hubs service, called Event Hubs Capture.

#### Event Hubs Capture
With Azure Event Hubs Capture, streaming data is automatically delivered to an Azure Blob storage or Azure Data Lake storage account for continued processing with services like Data Lake Analytics, Data Factory or HD Insights

* Destinations are Azure Blob Storage and Azure Data Lake Storage
* The event data is stored using the Avro file format

#### Amazon Kinesis Firehose
Amazon Kinesis Firehose is a service designed to be the easist way to load streaming data into data stores and analytics tools on the AWS platform. It can forward streaming data to Amazon S3, Amazon Redshift, Amazon Elasticsearch and Splunk

* Destinations are Amazon S3, Amazon Redshift (via S3 and Redshift `COPY`), Amazon Elasticsearch and Splunk
* Event data can be transformed before being sent for persistence using AWS Lambda functions
* Record size rounded up to nearest 5KB when billed

#### Buffering
Both Event Hubs Capture and Kinesis Firehose has configurable buffering, offering you the capability to specify when data should be persisted using both time and size windows. Event Hubs offer a larger Buffer Size but except from that they're very similar

|             | Firehose | Event Hubs Capture |
|-------------|----------|--------------------|
| Buffer size | 1 - 128 MB | 10 - 500 MB |
| Buffer window | 60 - 90 Sec | 1 - 15 min (60 - 900 Sec)|

#### Pricing
Let's use Amazon's example
> 1,000 records of streaming data per second for a month, each record 3KB in size to Firehose/Event Hubs Capture to be loaded into Amazon S3/Azure Blob Storage

As you can see in below calculations, the monthly charges for this scenario is more or less the same for both services.

Key differentiator for pricing is that Amazon charge by number of events and actual amount of ingested data while Microsoft charge for number of events and capture units.

##### Firehose
Record size of 3KB rounded up to 5KB.  
Data ingested (GB per sec) = (1,000 records / sec * 5KB/record) / 1,048,576 KB/GB = 0.004768 GB/sec  
Data ingested (BG per month) = 30days/month * 86,400 sec/day * 0.004768 GB/sec = 12,359.62 GB/month  
The price in US-East is $0.029 per GB of Data Ingested for the first 500 TB/month  
Monthly charges = 12,359.62 GB * $0.029/GB = __$358__

##### Event Hubs Capture
1000 events/sec = 2592M events /month * $0.028 = $73  
3 throughput units * $0.030 / hour  = $280 (inc Capture, $0.10/hr/unit)  
Monthly charges = $280 + $73 = __$353__

### Streaming Applications
Streaming applications consume the incoming events that are, in this case, being ingested by either Kinesis Firehose, Kinesis Streams or Azure Event Hubs. The first question might be which of the Kinesis services that you should pick if you'd to go with Amazon. 

#### Kinesis Firehose or Kinesis Data Streams?
As described before, Kinesis Firehose is primarily designed to allow you to easily ingest large amounts of events of AWS. Firehose will just receive the data and dump it into whatever destination you choose, and it'll scale according to your needs automatically.

Kinesis Data Streams on the other hand, gives you more control over provisioned capacity but will also require you or your app to scale the service. The most significant difference from a capability perspective is probably that Kinesis Data Streams keep the events in the stream (thus a stream..), allowing you to read events multiple times depending on your scenario.

Kinesis Firehose is the simpler option, if you just want to drop stuff into AWS storage.

#### Throughput
Both Kinesis Data Streams and Azure Event Hubs are designed to handle insanely large numbers of events but it's up to you to provision enough capacity according to your needs. Both services offers scale units which comes with a given throughput. Kinesis Data Streams calls this _shards_ while Event Hubs refers to it as _throughput units_. Both offer 1MB/s ingress and 2MB/s egress per scale unit.

| | Ingress | Egress |
|-|---------|--------|
| Event Hubs | 1MB/s | 2MB/s |
| Kinesis Data Streams | 1MB/s | 2MB/s|

(per scale unit)

Per default, Azure allows you to provision up to 20 throughput units (additional available after contacting support or in the dedicated tier). Using the _inflate_ feature, Event Hubs can autoscale the number of throughput units based on actual load.

#### Message retention
Both Kinesis Data Stream and Azure Event Hubs provide configurable message retention value from 1 to 7 days, with 1 day as the default. 

For Event Hubs, events could be kept for longer in the data stream depending on the number of events being passed. It'll not clear individual events but rather remove chunks of events as a group once the group is big enough. Thus it's important to checkpoint the stream properly and don't rely on the event retention policy.

#### Pricing
For both services, you'll be charged for the capacity provisioned (i.e throughput units / shards) per hour and number of events. Amazon Kinesis Data Streams might look significantly cheaper at a first glance (50%) but they'll also charge based on event size using something called _PUT Units_. Each PUT Unit is 25 KB thus an event of 26 KB will consume 2 units. Since Event Hubs support up to 256 KB events, it might mean a 10x cost increase per message on Kinesis Data Streams and 5x compared to Event Hubs - something to keep in mind!

| | Message Size (Unit) | / 1M events | / throughput unit |
|-|---------------------|-------------|-------------------|
| Event Hubs | 256 KB | $0.028 | $0.030 |
| Kinesis Data Streams | 25 KB | $0.014 | $0.015 |

Going back to our example of 1,000 3KB events / second / month;

##### Event Hubs
1000 events/sec = 2592M events /month * $0.028 = $73  
3 throughput units * $0.030 / hour = $65  
Monthly charges = $73 + $65 = __$138__

##### Kinesis Data Streams (3KB)
1000 events/sec = 2592M events/month * $0.014 = $36  
3 shards * $0.015 /hr = $32  
Monthly charges = $36 + $32 = __$68__

Now, let's increase the event size to 128KB to consume more PUT Units in Kinesis Data Streams. Remember, cost for Event Hubs will remain the same since it allows for 256KB events.

##### Kinesis Data Streams (128KB)
128KB event size = 6 PUT Units
1000 events/sec = 2592M events/month * $0.014 * 6 Units = $218  
3 shards * $0.015 /hr = $32  
Monthly charges = $218 + $32 = __$250__

Even though it's a big increase compared to smaller events, it's still very low figures and will probably not influence any decision on Kinesis vs Event Hubs.

### Stream Analytics
If implemented correct, this is an area where you can differentiate from your competition by gaining valuable insights from real-time data. Both AWS and Azure contains fully managed consumers which greatly simplifies the implementation effort and lets you focus on analyzing and reacting to the event data. 

AWS Kinesis Analytics and Azure Stream Analytics allow you to query the event stream using familiar SQL syntax. Select from the input stream and deliver the result to an output stream or another type of target. As you can see below, Azure Stream Analytics contains more built-in output types compared to AWS Kinesis Analysis which might cut development time for certain scenarios.

#### Inputs

| AWS Kinesis Analytics | Azure Stream Analytics |
|-----------------------|------------------------|
| Kinesis Data Streams  | Event Hubs             |
| Kinesis Firehose      | IoT Hub                |
|                       | Blob Storage           |

#### Outputs

| AWS Kinesis Analytics | Azure Stream Analytics |
|-----------------------|------------------------|
| Kinesis Data Streams  | Event Hubs             |
| Kinesis Firehose      | Data Lake Storage      |
| Lambda Function       | Function               |
|                       | SQL Database |
|						| Blob storage |
|						| PowerBI |
|						| Table Storage |
|						| Service Bus Queue |
|						| Service Bus Topic |
|						| Cosmos DB |

### Window functions
As expected, both services support window functions to let you bound queries using a window defined by time. With Azure Stream Analytics you'll always use the SQL `group by` clause to define what type of window you want to group by, while AWS Kinesis Analytics has different approaches depending on the type of window. Both have _Sliding_ and _Tumbling_ window types. Stream Analytics has a third type, _Hoping_ which essentialy is a overlapping Tumbling window.

| AWS Kinesis Analytics | Azure Stream Analytics |
|-----------------------|------------------------|
| Sliding         | Sliding         |
| Tumbling        | Tumbling        |
|                 | Hopping          |

### Deployment targets
Azure Stream Analytics jobs can be deployed to the edge, meaning field gateways and similar. This can be a critical capability for time sensitive operation where you for instance in a factory don't have cloud connectivity or can't afford the latency of the roundtrips to a cloud service. 
[See how Sandvik Coromant use Stream Analytics jobs on the edge](https://blogs.microsoft.com/iot/2016/09/12/azure-iot-suite-helps-sandvik-coromant-stay-on-cutting-edge-within-digital-manufacturing)

I haven't found any docs from Amazon indicating that this would be possible with AWS Kinesis Analytics.

### Pricing
Both Amazon and Microsoft charge by _Streaming Unit_ consumed. Amazon is more transparent on how much capacity you get for one unit which is 4GB memory and 1 vcpu compute, but it's still hard to know what that actually will be sufficient for. It comes down to how much memory and compute your queries consume and can only be determined by monitoring your jobs.

Both carge $0.11 per streaming unit per hour.

## Limits and Quotas
* [AWS Kinesis Analytics Limits](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/limits.html)
* [ AWS Kinesis Data Streams Limits](https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)
* [AWS Kinesis Firehose Limits](https://docs.aws.amazon.com/firehose/latest/dev/limits.html)
* [Azure Stream Analytics Limits](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#stream-analytics-limits)
* [Azure Event Hubs Limits](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#event-hubs-limits)

## Conclusion and Considerations
Looking at core capabilities both AWS Kinesis and Azure Event Hubs (with Stream Analytics) are very similar and you would probably be able to use any of them for your scenario.

That said, if you need to run analytics on data streams close to where they're emitted (like within a factory or in a car), Azure Stream Analytics seems to be the only option as of now.

* __Run on the edge__  - Azure Stream Analytics
* __Log shipping with simple client__ - AWS Firehose agent
* __Large messages__ - keep an eye on message cost on AWS Kinesis Data Streams
* __Interopability__ - Azure Event Hubs offer AMQP support while AWS Kinesis has their own thing over HTTP
* __Existing investments and knowledge__ - Azure vs AWS

## References and additional resources
* [Event Hubs Capture](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-capture-overview)
* [Event Hubs Capture in the portal](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-capture-enable-through-portal)
* [Kinesis Firehose getting started](https://aws.amazon.com/kinesis/data-firehose/getting-started/)
* [Firehose overview](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html)
* [Firehose Redshift COPY](https://docs.aws.amazon.com/redshift/latest/dg//r_COPY.html)
* [Firehose data transformation](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html)
* [Event Hubs Pricing](https://azure.microsoft.com/en-us/pricing/details/event-hubs/)
* [Kinesis Firehose pricing](https://aws.amazon.com/kinesis/data-firehose/pricing/)
* [Event Hubs features](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-features)
* [Event Hubs data retention](https://blogs.msdn.microsoft.com/servicebus/2015/03/09/data-retention-in-event-hubs/)
* [Kinesis Data Streams Data Retention](https://docs.aws.amazon.com/streams/latest/dev/kinesis-extended-retention.html)
* [Event Hubs Quotas](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-quotas)
* [Kinesis Analytics Overview](https://aws.amazon.com/kinesis/data-analytics/)
* [Kinesis Analytics Outputs](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/how-it-works-output.html)
* [Stream Analytics Overview](https://docs.microsoft.com/en-us/azure/stream-analytics/)
* [Stream Analytics Outputs](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-define-outputs)
* [Stream Analytics Window Functions](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions)
* [Kinesis Analytics Windowed SQL](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/windowed-sql.html)
* [Kinesis Analytics Pricing](https://aws.amazon.com/kinesis/data-analytics/pricing/)
* [Stream Analytics Streaming Units]( https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-streaming-unit-consumption)

