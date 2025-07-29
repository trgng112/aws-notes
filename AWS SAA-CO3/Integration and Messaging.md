- When we start deploying multiple applications, they will inevitably need to communicate with one another
- There are two patterns of application communication
- ![[Pasted image 20250729182723.png]]
- Synchronous between applications can be problematic if there are sudden spikes of traffic
- what if you need to suddenly encode 1000 videos but usually its 10?
- it that case, its better to **decouple** your applications,
	- Using SQS: queue model
	- Using SNS: pub/sub model
	- using Kinesis: real-time streaming model
- These services can scale independently from our application!


## SQS
### Whats a queue?
![[Pasted image 20250729182930.png]]
- Oldest offering (over 10 years old)
- Fully managed service, used to **decouple applications**
- Attributes:
	- Unlimited throughput, unlimited number of messages in queue
	- Default retention of messaged: 4 days, maximum of 14 days
	- Low latency (<10ms on publish and receive)
	- Limitation of 256kb per message sent
- Can have duplicate messages (at least once delivery ,occasionally)
- Can have out of order messages (best effort ordering)

### Producing Messages
- Produce to SQS using SDK (SendMessage API)
- the message is persisted in SQS until a consumer deletes it
- Message retention: default 4 days, up to 14 days
- Example: send an order to be processed
	- Order id
	- customer id
	- any attribute
	- SQS standard: unlimited throughput
	- ![[Pasted image 20250729183208.png]]

### Consuming Messages
- Consumers (running on EC2 instances, servers, AWS Lambda)
- Poll SQS for messages (receive up to 10 messages at a time)
- Process the messages (example: insert the message into an RDS database)
- Delete the message using DeleteMessage API
- ![[Pasted image 20250729183304.png]]

### Multiple EC2 instances consumers
- Consumers receive and process messages in parallel
- At least once delivery
- Best-effort message ordering
- Consumers delete messages after processing them
- We can scale consumers horizontally to improve throughput of processing
- ![[Pasted image 20250729183428.png]]

### SQS with Auto Scaling Group ASG
![[Pasted image 20250729183500.png]]
If the load is too big, some transactions may be lost
SQS as a buffer to database writes
![[Pasted image 20250729185116.png]]
SQS to **decouple** between application tiers
![[Pasted image 20250729185143.png]]

### SQS to **decouple** between application tiers
- ![[Pasted image 20250729183551.png]]

### Security
- Encryption:
	- In flight encryption using HTTPS API
	- At-rest encryption using KMS keys
	- Client-side encryption if the client wants to perform encryption/decryption itself
- Access Controls: IAM policies to regulate access to the SQS API
- SQS Access Policies (similar to S3 bucket policies)
	- Useful for cross-account access to SQS queues
	- Useful for allowing other services (SNS,S3....) to write to an SQS queue
### Message Visibility Timeout
- After a message is polled by a consumer, it becomes **invisible** to other consumers
- By default, the visibility timeout is **30 seconds**
- That means the message has 30 seconds to be processed
- After the message visibility timeout is over, the message is 'visible' in SQS
- ![[Pasted image 20250729184220.png]]
- If a message is not processed within the visibility timeout, it will be processed **twice**
- A consumer could call the **ChangeMessageVisibility** API to get more time
- if visibility timeout is high (hours) and consumer crashed, re-processing will take time
- if visibility timeout is too low (seconds), we may get duplicates

### Long Polling
- When a consumer request messages from the queue, it can optionally wait for messages to arrive if there are none in the queue
- This is called Long Polling
- **LongPolling decreases the number of API calls made to SQS while increasing the efficiency and reducing latency of your application**
- The wait time can be between 1 sec to 20 sec (20 sec preferable)
- Long polling is perferable to short polling
- Long Polling can be enabled at the queue level or at the api level using **WaitTimeSeconds**
![[Pasted image 20250729184714.png]]
### FIFO Queue
- FIFO = First in First Out (ordering of messages in the queue)
- Limited throughput: 300msg/s without batching, 30000 msg/s with 
- Exactly-once send capability (by removing duplicates using Deduplication ID)
- Messages are processed in order by the consumer
- **Ordering** by Message Group ID (all messages in the same group are ordered) - mandatory parameter
- ![[Pasted image 20250729184814.png]]




## SNS - Simple Notification Service
- What if you want to send one message to many receivers?
- ![[Pasted image 20250729185244.png]]
- The "event producer" only sends message to one SNS topic
- As many "event receivers" (subscriptions) as we want to listen to the SNS topic notificaitons
- Each subscriber to the topic will get all the messages (note: new feature to filter messages)
- Up to 12,500,500 subscriptions per topic
- 100,000 topics limit
- ![[Pasted image 20250729185412.png]]

### SNS integrates with a lot of AWS services
- Many AWS services can send data directly to SNS for notifications
- ![[Pasted image 20250729185447.png]]

### How to publish
- Topic publish (using the SDK)
	- Create a topic
	- Create a subscription (or many)
	- Publish to the topic
- Direct publish (for mobile apps SDK)
	- Create a platform application 
	- Create a platform endpoint
	- Publish to the platform endpoint
	- Works with Google GCM,AppleAPNS, Amazon ADM...

### Security
- Encryption:
	- In flight encryption using HTTPS API
	- at rest encryption using KMS keys
	- Client side encryption if the client wants to perform encryption/decryption itself
- Access controls: IAM policies to the regulate access to the SNS API
- SNS Access Policies (similar to S3 bucket policies)
	- Useful for cross-account access to SNS topics
	- Useful for allowing other services (S3) to write to an SNS topic

### SNS + SQS: Fanout
- Push once in SNS, receive in all SQS queues that are subscribers
- Fully decoupled, do ddata loss,
- SQS allows for data persistence, delayed processing and retires of work
- ability of add more SQS subscribers over time 
- Make sure your SQS queue **Access policy** allows for SNS to write
- ![[Pasted image 20250729195558.png]]
- Cross-region delivery: works with SQS queue in other regions

#### Application: S3 Events to multiple Queues
- For the same combination of event type (object create) and prefix (images/) you can only have one S3 Event rule
- If you want to send the same S3 event to many SQS queues, use fan-out
- ![[Pasted image 20250729195743.png]]

#### Application: SNS to S3 through Kinesis Data Firehose
- SNS can send to Kinesis and therefore we can have the following solution
- ![[Pasted image 20250729195816.png]]

#### FIFO Topic
- FIFO = First in First out (ordering of messages in the topic)
- Similar features as SQS FIFO:
	- Ordering by Message Group ID (all messages in the same group are ordered)
	- Deduplication using Deduplication ID or Content Based Deduplication
- **Can have SQS Standard and FIFO queues as subscribers** 
- Limited throughput (same as SQS FIFO)
- ![[Pasted image 20250729195939.png]]
- in case you need fanout + ordering + deduplication
- ![[Pasted image 20250729200008.png]]

#### Message FIltering
- JSON policy used to filter messages sent to SNS topic subscriptions
- If a subscription doesn't have a filter policy, it receives every message
- ![[Pasted image 20250729200100.png]]
- 

## Kinesis Data Streams
- Collect and store streaming data in **real-time**
- ![[Pasted image 20250729200443.png]]
- Retention between upto 365 days
- Ability to reprocess (replay) data by consumers
- Data cant be deleted from Kinesis (until it expires)
- Data up to 1MB (typical use case is lot of "small" **real-time data**)
- Data ordering guarantee for data with the same Partition ID
- At-rest KMS encryption, in-flight HTTPS encryption
- Kinesis Producer Library (KPL) to write an optimized producer application
- Kinesis Client Plibrary (KCL) to write an optimized consumer application

### Capacity mode
- Provisioned mode
	- Choose number of shards
	- Each shard get 1MB/s in (or 1000 records per second)
	- Each shard gets 2mb/s out
	- Scale manually to increase or decrease the number of shards
	- you pay per shard provisioned per hour
- On Demand mode:
	- No need to provision or manage the capacity
	- Default capacity provisied (4MB/s in or 4000 records per second)
	- Scales automatically based on observed throughput peak during the last 30 days
	- Pay per stream per hour & data in/out per GB


## Data Firehose
![[Pasted image 20250729201340.png]]

- Fully managed service
	- Redshift/S3/OpenSearch Service
	- 3rd partgy: Splunk/mongo db/ datadog
	- Custom HTTP endpoint
- Automatic scaling, serverless, pay for what you use
- **Near Real-Time** with buffering capability based on size/time
- Supports CSV,JSON ,Parquet,Avro,Raw text, binary data
- Conversion to Parquet/ORC, compressions with gzip/snappy
- Custom data transformations using AWS Lambda (CSV to JSON)
- 
### Kinesis Data streams vs Data Firehose
- Kinesis:
	- Streaming data collection
	- Producer and consumer code
	- Real time
	- Provisioned / on demand mode
	- Data storage up to 365 days
	- replay capability
- Firehose:
	- Load streaming data into s3 / reshift/ opensearch/3rd party/custom http
	- Fully managed
	- Near real time
	- Automatic scaling
	- no data storage
	- doesn't support replay capability

## SQS vs SNS vs Kinesis
![[Pasted image 20250729202138.png]]


## Amazon MQ
- SQS SNS are "cloud-native" services: proprietary protocols from AWS
- Traditional applications running from on premises may use open protocols such as MQTT, AMQT, STOM, Openwire,WSS
- When migrating to the cloud, instead of re-engineering the application to use SQS and SNS, we can use AMazon MQ
- **Amazon MQ is a managed message broker service for RabbitMQ, ActiveMQ**
- Amazon MQ doesn't scale as much as SQS, SNS
- MQ runs on server, can run in Multi-AZ with failover
- MQ has both queue features (SQS) and topic feature (SNS)
- ![[Pasted image 20250729202359.png]]
- 