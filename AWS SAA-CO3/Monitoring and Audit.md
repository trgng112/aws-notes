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

### Cloudwatch Agent and CLoudwatch logs agents
- By default, no logs from your EC2 machine will go to CW
- You need to run a CW agent on EC2 to push the log files you want
- Make sure IAM permissions are correct
- CW agents can be setup on-premises too 
- ![[Pasted image 20250805163803.png]]
- For virtual Servers: (ec2 instances, on premise servers)
- **CW logs Agent**
	- Old version of the agent
	- Can only send to CW logs
- **CW Unified Agent**
	- Collect additional system-level metrics such as RAM, processes, etc,...
	- Collect logs to send to CW Logs
	- Centralized configuration using SSM Parameter store
- **Unifield Agent Metrics**
	- Collected directly on your Linux server/ EC2 instance
		- CPU 
		- Disk metrics
		- RAM
		- Netstat
		- Processes
		- Swap SPace
	- Reminder: out of the box metrics for Ec2 - disk, cpu, network

## CW Alarms
- Alarms are used to trigger notifications for any metric
- Various options (sampling,%, max, min) 
- Alarm states:
	- OK
	- INSUFFICIENT_DATA
	- ALARM
- Period:
	- Length of time in seconds to evaluate the metric
	- High resolution custom metrics: 10,30 secs or multiples of 60 secs

### Alarm Targets
- Stop, terminate, reboot, or recover an EC2 instance
- Trigger Auto Scaling Action
- Send notification to SNS (from which you can do pretty much anything)
- ![[Pasted image 20250805164211.png]]
### Composite Alarms
- CW Alarms are on a single metric
- **Composite alarms are monitoring the states of multiple other alarms**
- AND and OR conditions
- Helpful to reduce 'alarm noise' by creating complex composite alarms
- ![[Pasted image 20250805164319.png]]

### EC2 Instance Recovery
- Status check:
	- Instance status = check the EC2 VM
	- System status = check the underlying hardware
	- Attached EBS status = check attached EBS volumes
	- Recovery: same Private, Public,Elastic IP, metadata, placement group
	- ![[Pasted image 20250805164427.png]]

### Good to know
- Alarms can be created based on CW Logs Metrics Filters
- To test alarms and notifications, set the alarm state to Alarm using CLI
- ![[Pasted image 20250805164508.png]]

## EventBridge
- Schedule: cron jobs (scheduled scripts)
- Event Pattern: Event rules to react to a service doing something
- Trigger Lambda functions, send SQS/SNS messages
- ![[Pasted image 20250805164835.png]]
- ![[Pasted image 20250805164943.png]]
- ![[Pasted image 20250805165032.png]]
- Event buses can be accessed by other AWs accounts using Resource-based Policies
- You can **archive events (all/filter) sent to an event bus (indefinitely or set period)**
- Ability to **replay archived events**

### Schema Registry
- EventBridge can analyze the events in your bus and infer the SChema
- The Schema Registry allows you to generate code for your app, that will know in advance how data is structured in the event bus
- Schema can be versioned

### Resource-based policy
- Manage Permissions for a specific event bus
- Example: allow/deny events from another AWS account or AWS region
- Use case: aggregate all events from your AWS org in a single AWS account or AWS region
- ![[Pasted image 20250805165313.png]]


## Cloudwatch Insights and Operational Visibility

### CW Container Insights
- Cloolect , aggregate, summarize metrics and logs from containers
- Available for containers on
	- Elastic Container Service - ECS
	- Elastic Elastic Kubernetes Service - EKS
	- Kubernetes platforms on EC2
	- Fargate (ECS and EKS)
- **In EKS and Kubernetes, CW insights is using a containerized version of the CW Agent to discover containers**

### CW Lambda Insights
- Monitoring and troubleshooting solution for serverless apps running on Lambda
- Collects, aggregates, and summarizes system-level metrics including CPU time, memory, disk and network
- Collects, aggregates, and summarizes diagnostic information such as cold starts and Lambda worker shutdowns
- Lambda Insights is provided as a lambda layer

### CW Contributor Insights
- Analyze log data and create time series that display contributor data
	- See metrics about the top N contributors
	- The total number of unique contributors and their usage
- This helps you find top talkers and understand who or what is impacting system performance.
- Works for any AWS generated Logs (VPC,DNS,...)
- For example: you can find bad host, **identify the heaviest network users** or find the URLs that generate the most errors.
- You can build your rules from scratch, or you can also use sample rules that AWS has created - **leverages your CW logs**
- CW also provides built in  rules that you can use to analyze metrics from other AWS services
- ![[Pasted image 20250805170300.png]]

### CW Application Insights
- **Provides automated dashboards that show potential problems with monitored applications, to help isolate ongoing issues**
- Your apps run on EC2 instances with select technologies only (java, .net, microsoft IIS Web server, dbs)
- And you can use other AWS resources like EBS RDS ELB ASG Lambda SQS DynamoDB,..
- Powered by SageMaker
- Enhanced visibility to app health to reduce the time to trouble shoot and repair the app
- Findings and alerts are sent to EventBridge and SSM OpsCenter

### Summary
- CW Container Insights
	- ECS, EKS, Kubernetes of EC2, Fargate, needs agents for Kubernetes
	- Metrics and Logs
- CW Lambda Insights
	- Detailed metrics to troubleshoot serverless applications
- CW Contributor Insights
	- Find TOP-N contributors through CW Logs
- CW apps insights
	- Automatic dashboard to troubleshoot your app and related aws services
- 
## CloudTrail
- Provides governance, compliance and audit for you AWS Account
- CloudTrail is enabled by default
- Get an history of events /API calls made within your AWS account :
	- Console
	- SDK
	- CLI
	- AWS services
- Can put logs from CloudTrail into CW logs or S3
- **A trail can be applied to All Regions (default) or a single Region**
- If a resource  is deleted in AWS, check Cloudtrail first
### CloudTrail Events
- Management Events:
	- Operations that are performed on resource in your AWS account
	- Example:
		- Configuring security (IAM AttachRolePolicy)
		- Configuring rules for routing data (EC2 CreateSubnet)
		- Setting up lgoging (CloudTrail CreateTrail)
	- By default: tgrails are configured to log management events
	- Can separate Read Events (don't modify resources) from WRite Events (may modify resources)
- Data Events:
	- By default, data events are not logged (cuz high volume operations)
	- S3 object level activity (GetObject,DeleteObject,PutObject): can separate Read and Write Events
	- Lambda function execution activity (Invoke API)

### CloudTrail Insights Events
- Enable CloudTrail insights to detect unusual activity in your account
	- Inaccurate resource provisioning
	- hitting service limits
	- burst of AWS IAM actions
	- Gaps in periodic maintenance activity
- Cloudtrail insights analyzes normal management events to create a baseline
- And then **continuously analyzes write events to detect unusual patterns**
	- Anomalies appear in the CloudTrail console
	- Event is sent to S3
	- EventBridge event is generated (for automation needs)
	- ![[Pasted image 20250805184915.png]]

### Events retention
- Events are stored for 90 days in CloudTrail
- To keep events beyond this period, log them to S3 and use Athena
- ![[Pasted image 20250805184953.png]]

### EventBridge - CloudTrail 
![[Pasted image 20250805185119.png]]
![[Pasted image 20250805185221.png]]

## AWS Config
- Helps with auditing and recording compliance of your AWS resources
- Helps record configurations and changes over time
- Questions that can be solved by AWS Config:
	- Is there unrestricted SSH access to my SGs?
	- Do my buckets have any public access?
	- How has my ALB config changed over time?
- You can receive alerts (SNS notifications) for any changes
- AWS Config is a per region service
- Can be aggregated across regions and accounts
- Possibility of storing the config data into S3 (analyzed by Athena)

### Config Rules
- Can use AWS managed config rules (over 75)
- Can make custom config rules (must be defined in Lambda)
	- Ex: evaluate if each EBS disk is of type gp2
	- ex: eval if each ec2 instance is t2.micro
- Rules can be evaluated / triggered:
	- For each config change
	- And/or: at regular time intervals
- **AWS Config Rules does not prevent actions from happening (no deny)**
- Pricing: no free tier, $0.003 per config item recorded per region, $0.001 per config rule eval per region

### Config Resource
![[Pasted image 20250805191113.png]]

### Config Rules - remediations
- Automate remediation of non compliant resources using SSM Automation Documents
- Use AWS Managed Automation Documents or create custom Automation Documents
	- Tip: You can create custom Automation Documents that invokes Lambda function
- You can set Remediation retries if the resource is still non-compliant after auto remediation

### Config rules - notification
![[Pasted image 20250805191346.png]]


## Cloudwatch vs cloudtrail vs config
- CloudWatch
	- Performance monitoring (metrics, CPU, network, etc) & dashboards
	- Events & Alerting
	- Log Aggregation & Analysis
- CloudTrail
	- Record API calls made within your Account by everyone
	- Can define trails for specific resources
	- Global service
- Config
	- Record config changes
	- Eval resources against compliance rules
	- Get timeline of changes and compliance

### For an ELB
- CW:
	- Monitoring incoming connections metric
	- Visualize error codes as a % over time
	- Make a dashboard to get an idea of your load balancer performance
- Config:
	- Track SG rules for the LB
	- Track config changes for the LB
	- Ensure an SSL cert is always assigned to the LB (compliance)
- CloudTrail:
	- Track who made any changes to the LB with API calls