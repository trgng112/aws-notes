## Athena
- **Serverless** query service to analyze data stored in S3
- Uses standard SQL langauge to query the files (built on Presto)
- Supports CSV,JSON, ORC, Avro, Parquet
- Pricing: $5.00 per TB of data scanned
- Commonly used with Amazon Quicksight for reporting/dashboards
- **Use cases**: Business intelligence/ analytics/reporting, analyze and query VPC Flow logs, ELB logs, CloudTrail trails  
- ![[Pasted image 20250802132257.png]]

### Performance improvement
- Use **columnar data** for cost-savings (less scan)
	- Apache Parquet or ORC is recommended
	- Hude performance improvement
	- Use Glue to convert your data to Parquet or ORC
- **Compress data** for smaller retrieval (bzip2,gzip,lz4,snappy,zlip,zstd)
- Partition datasets in S3 for easy querying on virtual colums 
	- ![[Pasted image 20250802132450.png]]
- Use larger files > 128MB to minimize overhead

### Federated Query
- Allows you to run SQL queries across data stored in relational ,non-relational, object, and custom data sources (AWS or on premises)
- Uses Data Source Connectors that run on AWS Lambda to run Federated Queries (Cloudwatch logs, dynamodb, RDS)
- ![[Pasted image 20250802132610.png]]
- Store the results back in S3


## Redshift
- Redshift is based on PostgreSQL, but it's not used for OLTP
- It's OLAP - online analyitcal processing (analytics and data warehousing)
- 10x better performance than other data warehouses, scales to PBs of data
- **Columnar** storage of data (instead of row based) & parallel query engine
- Two modes: Provisioned Cluster or Serverless cluster
- Has a SQL interface for performing the queries
- BI tools such as Amazon Quicksight or Tableau inegrate with it
- **vs Athena**: Faster queries / joins  /aggregations thanks to indexes

### Cluster
- Leader node: for query planning, results aggregation
- Compute node: for performing the queries, send results to leader
- Provisioned mode: 
	- Choose instance types in advance
	- Can reserve instances for cost savings
- ![[Pasted image 20250802133155.png]]

### Snapshots & DR
- Redshift has Multi AZ mode for some clusters
- Snapshots are point in time backups of a cluster, stored internally in S3
- Snapshot are incremental (only what has changes is saved)
- You can restore a snapshot into a new cluster
- Automated: every 8 hrs, every 5GB or on a schedule. Set retention
- Manual: snapshot  is retained until you delet eit
- **You can configure Redshift to automatically copy snapshots (automated or manual) of a cluster to another AWS region**
- ![[Pasted image 20250802133327.png]]
- ![[Pasted image 20250802133420.png]]

### Redshift Spectrum
- Query data that is already in S3 without loading it
- **Must have a Redshift cluster available to start the query**
- The query is then submitted to thousands of Redshift Spectrum nodes
- ![[Pasted image 20250802133528.png]]
## OpenSearch (ElasticSearch)
- Successor to Amazon ElasticSearch
- In DynamoDB, queries only exists by primary key or indexes...
- With OpenSearch, you can search any fields, even partially matches
- It's common to use OpenSearch as a complement to another db
- Two modes: managed cluster or serverless cluster
- Does not natively support SQL (can be enabled via a plugin)
- Ingestion from kinesis data firehose, aws iot, cloudwatch logs
- security through cognito & IAM ,KMS encryption,TLS
- Comes with OpenSearch Dashboards (viz)
- ![[Pasted image 20250802133732.png]]
- ![[Pasted image 20250802133758.png]]
- ![[Pasted image 20250802133828.png]]
## EMR

- EMR stands for "Elastic MapREduce"
- EMR helps creating **Hadoop clusters (Big Data)** to analyze and process vast amount of data
- The clusters can be made of **hundreds of EC2 instances**
- EMR comes bundled with Apache Spark, HBase, Presto, Flink...
- EMR takes care of all the provisioning and configuration
- Auto-scaling and integrated with Spot instances
- **Use cases: data processing, machine learning, web indexing, big data**

### Node types & purchasing
- Master node: manage the cluster, coordinate, manage health - long running
- Core node: Run tasks and store data - long running
- Task node (optional): Just to run tasks - usually Spot
- Purchasing options
	- On demand : reliable ,  predictable, wont be terminated
	- Reserved (min 1 year): cost savings (EMR will automatically use if available)
	- Spot instances: cheaper, can be terminated, less reliable
- Can have long-running cluster, or transient (temp) cluster

## Quicksight

- Serverless machine learning powered business intelligence service to create interactive dashboards
- Fast automatically scalable, embeddable,with per session pricing
- Use cases:
	- Business analytics
	- Building data viz
	- Perform ad hoc analysis
	- Get business insights using data
- Integrated with RDS, Aurora, Athena, Redshift, S3...
- **In-memory computation using SPICE** engine if data is imported into QuickSight 
- Enterprise edction: possibility to setup **Column-Level security** CLS
-  ![[Pasted image 20250802134302.png]]

### Integrations
- ![[Pasted image 20250802134348.png]]

### Dashboard & Analysis
- Define Users (standard versions) and Groups (enterprise version)
	- These users  & groups only exist within QuickSight, not IAM!!
- A dashboard
	- Is a read-only snapshot of an analysis that you can share
	- preserves the configuration of the analysis (filtering, parameters, controls, sort)
- **You can share the analysis or the dashboard with Users or Groups**
- To share a dashboard, you must first publish it
- Users who see the dashboard can also see the underlying data

## Glue
- Managed **extract,transform,and load (ETL)** service
- Useful to prepare and transform data for analytics
- Fully serverless service
- ![[Pasted image 20250802134653.png]]

### Convert data into Parquet format
- ![[Pasted image 20250802134735.png]]

### Glue Data Catalog: catalog of datasets
- ![[Pasted image 20250802134815.png]]

### Things to know at a high level
- **Glue Job Bookmarks** prevent re-processing old data
- **Glue Databrew**: clean and normalize data using pre-built transformation
- **Glue Studio** : new GUI to create, run and monitor ETL jobs in Glue
- **Glue Streaming ETL** (built on Apache Spark Structured Streaming): compatible with Kinesis Data Streaming, Kafka, MSK (managed Kafka)

## Lake Formation
- Data Lake = central place to have all your data for analytics purpose
- Fullly managed service that makes it easy to setup a data lake in days
- Discover, cleanse, transform, and ingest data into your Data Lake
- It automates many complex manual steps (collecting, cleansing, moving, cataloging data,...) and de-duplicate (using ML transforms)
- Combined structured and unstructured data in the data lake
- **Out of the box source blueprints**: S3, RDS, Relational & NoSQL DB...
- **Fine-grained Access Control for your apps (row and column level)**
- Build on top of AWS glue
- ![[Pasted image 20250802135219.png]]

### Lake Formation Centralized Permissions Example
![[Pasted image 20250802135311.png]]

## Amazon Managed Service for Apache Flink

-  Flink (Java, Scala or SQL) is a framework for processing data streams
- Run Any Apache Flink application on a managed cluster on AWS
	- Provisioned compute resources, parallel computation, automatic scaling
	- Application backups (implemented as checkpoints and snapshots)
	- Use any Apache Flink programming features to transform data
	- **Flink cannot read from Amazon Data Firehose**
	- ![[Pasted image 20250803211443.png]]

## MSK - Managed streaming for Apache Kafka
- Alternative for Amazon Kinesis
- Fully managed Apache Kafka on AWS
	- Allow you to create, update, delete clusters
	- MSK creates and manages Kafka brokers nodes & Zookeeper nodes for you
	- Deploy the MSK cluster in your VPC, multi AZ (up to 3 HA)
	- Automatic recovery from common Apache Kafka failures
	- Data is stored on EBS volumes for as long as you want
- **MSK Serverless**
	- Run Apache Kafka on MSK without managing the capacity
	- MSK automatically provisions resources and scales compute & storage

### Apache Kafka at a high level
- ![[Pasted image 20250803211756.png]]
- ![[Pasted image 20250803211846.png]]
### Amazon MSK Consumers
- ![[Pasted image 20250803211916.png]]

## Big data Ingestion Pipeline
- We want the ingestion pipeline to be fully serverless
- We want to collect data in real time
- We want to transform the data
- we want to query the transformed data using SQL
- The reports created using the queries should be in S3
- We want to load that data into a warehouse and create dashboards
- ![[Pasted image 20250803212145.png]]
- Iot core allows you to harvest data from IoT devices
- Kinesis is great for real time data collection
- Firehose helps with data delivery to S3 in near real time
- Lambda can help Firehose with data transformations
- S3 can trigger notifications to SQS
- Lambda can subscribe to SQS (we could have a connector from S3 to Lambda)
- Athena is a serverless SQL service and results are stored in S3
- The reporting bucket contains analyzed data and can be used by reporting tool such as AWS quicksight, redshift,...