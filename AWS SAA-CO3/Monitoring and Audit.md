## Cloudwatch Metrics
- provides metrics for every services in AWS
- Metric is a variable to monitor (CPU Utilization,NetworkLn...)
- Metrics belongs to namespaces
- Dimensions is an attribute of a metric (instance id , environment)
- up to 30 dim per metrics
- metrics have timestamps
- can create dashboard of metrics in cloudwatch
- create cloudwatch custom metrics ( for the ram for example)

### Cloudwatch metric streams
- Continually stream cloudwatch metrics to a destination of your choice **near real time delivery** and low latency.
	- Amazon Kinesis Data Firehose (and then its destinations)
	- 3rd party service provider : Datadog, dynatrace, splunk,....
	- Option to filter metrics to only stream a subset of them
	- ![[Pasted image 20250804185944.png]]
### Cloudwatch logs
- Log group: arbitrary name, usually representing an apps
- log stream: instances within apps/ log files/containers
- Can define log expiration policies (never expire, 1 day to 10 yrs)
- **Cloudwatch logs can send logs to**:
	- S3 (exports)
	- Kinesis data streams
	- data firehose
	- lambda
	- opensearch
- Logs are encrypted by default
- can setup kms based encryption with your own keys

#### Log source
- SDK, Cloudwatch logs agent, cloudwatch unified agent
- elastic beanstlak: collection of logs from apps
- ecs: collection from containers
- lambda: collection from function logs
- vpc flow logs: vpc specific logs
- api gateway
- cloudtrail based on filter
- route 53: log dns queries

#### Log Insights
- Search and analyze log data stored in CW logs
- Example: find a specific IP inside a log, count occurrences of ERROR in your logs
- Provides a purpose-built query language
	- Automatically discover fields from AWS services and JSON log events
	- Fetch desired event fields, filter based on conditions, calculate aggregate statistics, sort events, limit number of events
	- can save queries and add them to CW dashboards
- Can query multiple log groups in different AWs accounts
- It's a query engine, not a real time engine
#### S3 export
- Log data can take up to 12 hrs to become avail for export
- The API call is **CreateExportTask*
- Not near real time or real time... use Logs Subscription instead
#### Logs subscriptions
- get real time log events from CW logs for processing and analysis
- Send to kinesis data streams, data firehose or lambda
- **Subscription Filter** - filter which logs are events delivered to your destination
- ![[Pasted image 20250804190945.png]]
- ![[Pasted image 20250804191004.png]]
- ![[Pasted image 20250804191033.png]]