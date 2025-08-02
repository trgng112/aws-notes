[[RDS and Aurora and ElasticCache]] [[Lambda & DynamoDB & Serverless]]

## Choosing the right DB
- We have a lost of managed dbs on AWS to choose from
- Questions to choose
	- Read heavy, write heavy, balanced workload? throughput needs, will it change, does it need to scale or fluctutate during the day?
	- How much data to store and for how long? Will it grow? Average object size? how are they accessed?
	- Data durabililty? Source of truth for the data?
	- Latency requirements? Concurrent users?
	- Data model? how will you query the data? joins? structured?semi-structured?
	- Strong schema? more flexibility? reporting? search? RDBMS/NoSQL?
	- License cost? Swtich to cloud native DB such as aurora


## Database Types
- RDBMS (SQL) = RDS , Aurora - great for joins
- NOSQL db - no joins, no SQL = DynamoDB (Json), ElastiCache (key/value pairs), Neptune (Graphs), DocumentDB (for mongo db), Keyspaces (apache cassandra)
- Object Store: S3 (for big objects) / Glacier (for backups / archives)
- Data warehouse (=SQL analytics / BI): Redshift (OLAP), Athena , EMR
- Search: OpenSearch (JSON) - free text, unstructured searches
- Graphs: Amazon Neptune - display relationships between data
- Ledger: Amazon Quantum Ledger DB
- Time series: Amazon Timestream


## RDS
- Managed PostgresSQL/MySQL/Oracle/SQL Server/DB2/Maria DB?Custom
- Provisioned RDS instance size and EBS volume type & size
- Auto scaling capability for Storage
- Support for read replicas and multi az
- security through IAM, SGs KMS, SSL in transit
- Automated Backup with Point in time restore feature (up to 35 days)
- Manual DB Snapshot for longer term recovery
- Managed and Scheduled maintenance (with downtime)
- Support for IAM authentication, integration with secrets Manager
- RDS Custom for access to and customize the underlying instance (oracle & SQL server)
- Use case: store relational datasets, perform SQL queries, transactions

## Aurora
- Compatible API for PostgreSQL/ MYSQL, separation of storage and compute
- Storage: data is stored in 6 replicas, across 3 AZ- highly available, self-healing,auto scaling
- Compute: Cluster of DB instance across multiple AZ, auto-scaling of Read replicas
- Cluster: custom endpoints for writer and reader DB instances
- Same security/monitoring / maintenance feature as RDS
- Know the backup & restore options for Aurora
- **Aurora Serverless** -  for unpredictable / intermitten workloads, no capacity planning
- **Aurora Global**: up to 16DB Read Instances in each region < 1 second storage replication
- **Aurora Machine Learning**: perform ML using SageMaker & Comprehend on Aurora
- **Aurora Database Cloning** new cluster from existing one, faster than restoring a snapshot
- use case: same as RDS, but with less maintenance / more flexibility, performance, features

## Elastic Cache
- Managed Redis/Memcached (same as RDS but for caches)
- in memory data store, sub-millisecond latency
- select and elasticache instance type (cache.m6g.large)
- support for clustering (redis) and multi AZ, read replicas (Sharding)
- security through IAM , SGs , KMS , redis auth
- Backup / snapshot/point in time restore feature
- Managed and Scheduled maintenance
- **Requires some application code changes to be leveraged**
- Use case: key value store, frequent reads, less writes, cache result for db queries, store session data for websites, cannot use sql


## DynamoDB
- AWS proprietary tech, managed serverless no SQL db, millisecond latency
- capacity mode: provisioned capacity with optional auto-scaling or on-demand capacity
- Can replace elastiCache as a key/value store (storing session data with TTL feature)
- Highly available, multi AZ by default, read and writes are decoupled, transaction capability
- DAX cluster for read cache, microsecond read latency
- security,authentication and authorization is done through IAM
- Event Processing: DynamoDB streams to integrate with AWS lambda or kinesis data streams
- Global Table feature: active active setup
- automated backup up to 35 days with PITR (restore to new table) or on demand backups
- Export to S3 without using RCU within the PITR window, import from S3 without using WCU
- **Great to rapidly evolve schemas**
- Use case: serverless apps development (small documents 100s KB), distributed serverless cache

## S3
- key value store for objects
- great for bigger objects, not so great for many small objects
- serverless, scales infinitely, max object size is 5TB, versioning capability,
- Tiers: S3 Standard, S3 Infrequent Access, S3 Intelligent, S3 Glacier + lifecycle policy,
- Features: Versioning, encryption, Replication, MFA- Delete, Access Logs...
- Security: IAM , Bucket policies, ACL, Access points, object lambdas, cors, object/vault lock
- Encryption: SSE S3, SSE KMS, SSE C, client side, TLS in transit, default encryption
- Batch operations on objects using S3 Batch, listing files using S3 inventory
- Performance: multi-part upload, S3 transfer acceleration, S3 select
- Automation: S3 Event Notifications (SNS,SQS,Lambda,EvnetBridge)
- Use case: static files, key value store for big files, website hostings

## DocumentDB
- Aurora = postgresSQL/MySQL
- DocumentDB is the same for mongodb
- MongoDB is used to store,query,and index json data
- similar deployment concepts as aurora
- Fully managed, highly availab le with replication across 3AZ
- DocumentDB storage automatically grows in increments of 10gb
- automatically scales to workloads with millions of request per seconds


## Neptune
- Fully managed graph db
- A popular graph dataset would be a social network
	- User have friends
	- Post have comments
	- Comments have likes from users
	- Users share and like post...
- Highly available across 3 AZ, with up to 15 read replicas
- Build and run apps working with highly connected datasets- optimized for these complex and hard queries
- Can store up to billions of relations and query the graph with millisecond latency
- Highly available with replications across multiple AZs
- Great for knowledge graphs (wikipedia), fraud detection, recommendation engine, social network 
- ![[Pasted image 20250802130230.png]]


### Neptune streams
- **Real time ordered** sequence of every change to your graph data
- Changes are available immediately after writing
- **No duplicates, strict order**
- Streams data is accessible in an **HTTP Rest API**
- Use cases:
	- Send notifications when certain changes are made
	- Maintain your graph data synchronized in another data store (s3 , open search, elasti cache)
	- replicate data across regions in Neptune
## Keyspaces for Apache Cassandra
- Apache Cassandra is an open source NOSQL distributed database
- A manged Apache Cassandra - compatible database service
- Serverless, scalable, highly available, fully managed by AWs
- Automatically scale tables up/down based on the apps traffic
- Tables are replicated 3 times across multiple AZ
- Using the Cassandra Querry Language (CQL)
- Single digit millisecond latency at any scale, 1000s of requests per second
- Capacity: on demand and provisioned with auto scaling
- Encryption, backup, PITR up to 35 days
- use cases: store IoT devices info, time-series data,...

## Timestream
- Fully managed, fast, scalable, serverless **time series database**
- Automatically scales up/down to adjust capacity
- Store and analyze trillions of events per day
- 1000s time faster & 1/10th the cost of relational dbs
- Scheduled queries, multi-measure records, SQL compatibility
- Data storage tiering: recent data kept in memory and historical data kept in a cost optimized storage
- Built in time series analytics functions (helps you identify patterns in your data in near real-time)
- Encryption in transit and at rest
- use case: IoT apps, operational applications, real-time analytics

### Architeture
- ![[Pasted image 20250802130815.png]]

