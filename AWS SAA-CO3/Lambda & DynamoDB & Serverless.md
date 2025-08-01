## Lambda

- Easy Pricing
	- Pay per request and compute time
	- Free tier of 1mil AWS Lambda requests and  400,000 Gbs of compute time
- Integrated with the whole AWS suite of services
- Integrated with many programming languages
- Easy monitoring through Cloudwatch
- Easy to get more resources per functions (up to 10gb of ram)
- Increasing Ram will also improve cpu and network

### Language support
- node , python, java, C# .Net, Ruby,Custom Runtime API (community supported, example Rust or Golang0
- Lambda Container Image
	- The container image must implement the Lambda runtime API
	- ECS/Fargate is preferred for running arbitrary Docker images

### Lambda integrations 
### Main ones
- ![[Pasted image 20250731172929.png]]
### Thumbnail creation
- ![[Pasted image 20250731173004.png]]
### Serverless CRON job
- ![[Pasted image 20250731173032.png]]

### Pricing
- Pay per calls
	- First 1mil requests are free
	- 0.2$ per 1mil requests thereafter
- Pay per duration (in increment of 1ms)
	- 400,000 GB seconds of compute time per month if free
	- == 400,000 seconds if function is 1GB ram
	- == 3,200,000 seconds if function is 128Mb Ram
	- After that 1$ for 600,000 GB-seconds
- It is usually very cheap to run Lambda so its popular


### Lambda Limits - per region
- Execution:
	- Memory allocation : 128mb - 10gb (1mb increment)
	- Maximum execution time: 900 seconds (15 minutes)
	- Environment variables (4kb)
	- disk capacity in the 'function container' (in /tmp): 512Mb to 10Gb
	- Concurrency executions: 1000 (can be increased)
- Deployment:
	- Lambda function deployment size (compressed .zip):50Mb
	- Size of uncompressed deployment (code + dependencies) : 250mb
	- Can use the /tmp  directory to load other files at startup
	- Size of environment variables :4kb

### Lambda Concurrency and Throttling
- Concurrency limit: up to 1000 concurrent executions
-  Can set a **reserved concurrency** at the function level (=limit)
- Each invocation over the concurrency limit will trigger a 'Throttle'
- Throttle behavior:
	- If synchronous invocation => return ThrottleError - 429
	- If asynchronous invocation => retry automatically then go to DLQ

#### Concurrency Issue
- If you don't reserve concurrency, the following can happen
- ![[Pasted image 20250731174047.png]]
#### Concurrency and Asynchronous Invocations
- If the function doesn't have enough concurrency, available to process all events, additional requests are throttled
- For throttling errors (429) and system errors (500 series), Lambda returns the event to the queue and attempts to run the function again for up to 6 hrs
- The retry interval increases exponentially from 1 second after the first attempt to a maximum of 5 minutes
- ![[Pasted image 20250731174205.png]]

#### Cold Starts & provisioned concurrency
- **Cold start**
	- New instance => code is loaded and code outside the handler run (init)
	- if the init is large (code, dependencies , sdk) this process can take some time
	- First request served by new instances has higher latency than the rest
- **Provisioned Concurrency**
	- Concurrency is allocated before the function is invoked (in advance)
	- So the cold start never happens and all invocations have low latency
	- Application Auto Scaling can manage concurrency (schedule or target utilization)
- Note:
	- Cold start in VPC have been dramatically reduced in oct and nov 2019

#### Reserved and provisioned concurrency
![[Pasted image 20250731174424.png]]


### Lambda Snapstart
- Improves your lambda functions performance up to 10x at no extra cost for java, python & .net
- When enabled, function is invoked from a pre initialized state (no function init from scratch)
- When you publish a new version
	- Lambda init your function
	- Takes a snapshot of memory and disk state of the init function
	- snapshot is cached for low latency access
- ![[Pasted image 20250731174721.png]]

### Lambda Edge and CloudFront functions
- Many modern applications execute some form of the logic at the edge
- **Edge Functions**
	- A code that you write and attach to CloudFront distributions
	- Runs close to your users to minimize latency
- Cloudfront provides two types: **CloudFront Functions & Lambda@Edge**
-  You don't have to manage any servers, deployed globally
- Use case: customize the CDN content
- pay only for what you use
- fully serverless
#### Use cases
- ![[Pasted image 20250731174914.png]]

#### Cloudfront Functions
- Lightweight functions written in JS
- for high-scale, latency sensitive CDN customizations
- sub-ms startup times, millions of request/second
- used to change viewer requests and responses:
	- Viewer request: after cloudfront receives a request from a viewer
	- viewer response; before cloudfront forwards the response to the viewer
- Native feature of cloudfront (manage code entirely within Cloudfront)
-  ![[Pasted image 20250731175033.png]]


#### Lambda@edge
- Lambda functions written in Node or Python
- Scales to 1000s of requests/second
- Used to change CloudFront requests and responses:
	- Viewer Request - after CloudFront receives a request from a viewer
	- Origin Request - before CloudFront forwards the request to the origin
	- Origin Response - after CloudFront receives the response from the origin
	- Viewer response - before Cloudfront forwards the response to the viewer
- Author your functions  in one AWS region (us-east-1), then CloudFront replicates to its locations
- ![[Pasted image 20250731175228.png]]

### CloudFront functions vs lambda @edge
![[Pasted image 20250731175301.png]]
Use cases:
- CloudFront functions:
	- Cache key normalization
		- Transform request attributes to create an optimal cache key
	- Header manipulation
		- Insert/modify/delete HTTP headers in the request or response
	- URL rewrites or redirects
	- Request authentication & authorization
		- Create and validate user generated tokens JWT to allow/deny requests


### Lambda in VPC
- By default, your lambda function is launched outside your own VPC (AWS - VPC)
- Therefore it cannot access resources in your vpc (RDS,ElastiCache,ELB) 
- ![[Pasted image 20250801121850.png]]
- Define the VPC ID, the Subnets and Security groups
- Lambda will create an ENI (Elastic Network Interface) in your subnets
- ![[Pasted image 20250801121930.png]]

#### Lambda with RDS Proxy
- If Lambda functions directly access your db, they may open too many connections under high load
- RDS proxy
	- Improve scalability  by pooling and sharing DB connections
	- Improve availability by reducing by 66% the failover time and preserving connections
	- Improve security by enforcing IAM authentication and storing credentials in Secrets Manager
- **The Lambda function must be deployed in your VPC, because RDS Proxy is never publicly accessible**
- ![[Pasted image 20250801122113.png]]

### Invoking Lambda from RDS & Aurora
- Invoke lambda functions from within your db instance
- Allows you to process **data events** from within a database
- Supported for RDS for PostgreSQL and Aurora MySQL
- **Must allow outbound traffic to your Lambda function** from within your DB instance (Public, NAT GW,VPC Endpoints)
- **DB instance must have the required permissions to invoke the Lambda function** (Lambda Resource-based Policy & IAM Policy)
- ![[Pasted image 20250801122302.png]]

#### RDS Event Notifications
- Notifications that tells information about the DB instance itself (created,stopped, start)
- You don't have any information about the data itself
- Subscribe to the following event categories: **DB instance,DB snapshot,DB parameter group, DB security group, RDS Proxy, Custom Engine Version**
- Near real-time events (up to 5 minutes)
- Send notifications to SNS or subscribes to events using EventBridge
- ![[Pasted image 20250801122440.png]]

## DynamoDb

- Fully managed, highly available with replication across multiple AZs
- NoSQL database - with transaction support
- Scales to massive workloads, distributed database
- Millions of requests per seconds, trillions of row, 100s of TB of storage
- Fast and consistent in performance (single-digit millisecond)
- Integrated with IAM for security, authorization and administration
- Low cost and auto-scaling capabilities
- No maintenance or patching, always available
- Standard & IA (Infrequent Access) Table Class

### Basics
- DynamoDB is made of **Tables**
- Each table has a Primary Key ( must be decided at creation time)
- Each table can have an infinite number of items (=rows)
- Each item has attributes (can be added over time - can be null)
- Maximum size of an item is 400kb
- Data types supported are
	- Scalar Types - String, Number,Binary,Boolean,Null
	- Document Types - List, Map
	- Set Types - String Set, Number Set, Binary Set
- **Therefore, in DynamoDB you can rapidly evolve schemas**
- ![[Pasted image 20250801122837.png]]

### Read/Write capacity modes
- Controls how you manage your table's capacity (read/write throughput)
- Provisioned Mode  (default)
	- Specify the number of reads/writes per second
	- You need to plan capacity before hand
	- Pay for **provisioned** Read Capacity Unites (RCU) & Write Capacity Units (WCU)
	- Possibility to add auto scaling mode for RCU & WCU
- On demand mode
	- Read/writes automatically scale up/down with your workloads
	- No capacity planning needed
	- Pay for what you use, more expensive
	- Great for unpredictable workloads, steep sudden spikes
	- 

### Dynamo DB Accelerator (DAX)
- Fully managed, highly available, seamless in-memory cache for DynamoDb
- **Helps solve read congestion by caching**
- **Microseconds latency for cached data**
- Doesn't require application logic modifications ( compatible with existing DynamoDB APIs)
- 5 min TTL default
- ![[Pasted image 20250801123517.png]]

- DAX vs ElasticCache
- ![[Pasted image 20250801123538.png]]
### Stream Processing
- Ordered stream of item-level modifications (create/update/delete) in a table
- Use cases:
	- React to changes in real time (welcome email to users)
	- Real time usage analytics
	- Insert into derivative tables
	- Implement cross-region replication
	- Invoke AWS Lambda on changes to your DynamoDB table
- **DynamoDB streams**
	- 24 hrs retention
	- limited # of consumers
	- Process using AWS Lambda Triggers, or DynamoDB stream Kinesis adapter
- **Kinesis Data Streams (newer)**
	- 1 year retention
	- High # of consumers
	- Process using Lambda, Kinesis Data analytics, Kinesis data firehose, AWS Glue Streaming ETL
- ![[Pasted image 20250801123828.png]]

### Global Tables
- Make a DynamoDB table accessible with low latency in multiple regions
- Active-Active replication
- Applications can READ and WRITE to the table in any region
- Must enable DynamoDB Streams as a pre-requisite

### TTL
- Automatically delete items after an expiry timestamp
- Use cases: reduce stored data by keeping only current items, adhere to regulatory obligations, web session handling
- ![[Pasted image 20250801124030.png]]

### Backups for disaster recovery
- Continuous backup using point-in-time recovery (PITR)
	- Optionally enabled for the last 35 days
	- Point-in-time recovery to any time within the backup window
	- The recovery process creates a new table
- On demand backups
	- Full backups for long term retention, until explicitly deleted
	- Doesn't affect performance or latency
	- Can be configured and managed in AWS backup (enable cross-region copy)
	-  The recovery process creates a new table

### Integration with Amazon S3
- Export to S3 (must enable PITR)
	- Works for any point of time in the last 35 days
	- Doesn't affect the read capacity of your table
	- Perform data analysis on top of DynamoDB
	- Retain snapshots for auditing
	- ETL on top of S3 data before importing back into DynamoDB
	- Export in DynamoDB JSON or ION format
	- ![[Pasted image 20250801124324.png]]
- import from s3
	- Import CSV, DynamoDB JSON, ION format
	- Doesn't consume any write capacity 
	- ![[Pasted image 20250801124351.png]]

## API Gateway
- ![[Pasted image 20250801124445.png]]
- Lambda + Gateway: No infrastructure to manage
- Support for the WebSocket Protocol
- Handle API versioning (v1,v2)
- Handle different environments (dev,test,prod)
- Handle security (Authentication and Authorization)
- Create API keys, handle request throttling
- Swagger/Open API import to quickly define APIs
- Transform and validate requets and responses
- Generate SDK and API specs
- cache API responses

### Integrations High Level
- Lambda function
	- Invoke Lambda Function
	- Easy way to expose REST API backed by Lambda
- HTTP
	- Expose HTTP endpoints in the backend
	- Example: internal HTTP API on premise, ALB,
	- Add rate limiting, caching, user auth, API keys, 
- AWS services
	- Expose any AWS API through the gateway
	- Example: start an AWS Step function workflow,post a message to SQS
	- Add auth,deploy publicly, rate control
- ![[Pasted image 20250801124836.png]]

### Endpoint Types
- Edge Optimized (defaults) for Global clients
	- Requests are routed through the CloudFront Edge locations (improves latency)
	- The API Gateway still lives in only one region
- Regional:
	- For clients within the same region
	- Could manually combine with CloudFront (more control over the caching strategies and the distribution)
- Private:
	- Can only be accessed from your VPC using interface VPC endpoint (ENI)
	- Use a resource policy to define access


### Security
- User  Authentication through
	- IAM roles (internal apps)
	- Cognito (identity for external users- mobile users)
	- Custom Authorizer (own logic)
- Custom Domain Name HTTPS security through integration with AWS Certificate Manager (ACM)
	- If using edge-optimized endpoint, then the certificate must be in us-east-1 **
	- if using Regional endpoint, the certificate must be in the API Gateway region 
	- Must setup CNAME or A-alias record in Route 53


## Step Function
- Build serverless visual workflow to orchestrate your Lambda functions
- features:
	- Sequence, parallel, conditions, timeouts, error handling,...
- Can integrate with EC2, ECS, on premises servers, API Gateway, SQS queues
- Possibility of implementing human approval feature
- Use case: order fulfillment, data processing, web applications, any workflow
- ![[Pasted image 20250801132408.png]]

## AWS Cognito
- Give users an identity to interact with our web an mobile apps
- Cognito user Pools
	- Sign in functionality for apps users
	- integrate with API Gateway & ALB
- Cognito Identity Pools (federated Identity)
	- Provide AWS credentials to users so they can access AWS resources directly
	- Integrate with Cognito User Pools as an identity provider
- Cognito vs IAM: 'hundred of users','mobile users', 'authenticate with saml'

### Cognito User Pools (CUP) - User Features
- **Create a serverless database of user for your web and mobile apps**
- Simple login: username (email)/ password combination
- Password reset
- Email & Phone Number verification
- MFA
- Federated Identities : users from FB,GG,SAML
###  CUP Integrations
- CUP integrates with API Gateway and ALB
- ![[Pasted image 20250801132746.png]]

### CUP - Federated Identities
- Get identities for users so they obtain temp AWS credentials
- Users sources can be Cognito User Pools, 3rd party logins,
- Users can access AWS services directly or through API Gateway
- IAM policies applied to the credentials are defined in Cognito
- They can be customized based on the user_id for fine grained control
- Default IAM roles for authenticated and guest users

### Diagram
![[Pasted image 20250801132944.png]]
