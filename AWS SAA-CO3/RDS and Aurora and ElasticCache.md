### RDS
#### Amazon Relational Database Service
- Managed DB service that uses SQL as a query language
- Allow you to create dbs that are manage by AWS
	- Postgres
	- MySQL
	- MariaDB
	- ORacle
	- SQL server
	- Aurora (AWS Proprietary database)
- RDS is a managed service:
	- Automated provisioning, OS patching
	- Continuous backups and restore to specific timestamps (Point In Time Restore)
	- Monitoring dashboards
	- Read replicas for improved read performance
	- Multi AZ setup for Disaster REcovery
	- Maintenance windows for upgrades
	- Scaling capability (vertical and horizontal)
	- Storage backed by EBS
- But you can't SSH into your underlying instances
- Storage Auto Scaling
	- Helps you increase storage on your RDS DB instance dynamically
	- scales automatically -detects when you are running out of free db storage
	- Avoid manually scaling your db storage
	- You have to set Maximum Storage Threshold (maximum limit for DB storage)
	- Automatically modify storage if 
		- Free storage is less than 10% of allocated storage
		- Low storage lasts at least 5 mins
		- 6 hours have passed since last modification
		- Useful for applications with unpredictable owrkloads
		- Supports all RDS db engines 
		- ![[Pasted image 20250721203412.png]]
- RDS Read Replicas and multi AZ
	- Helps you scale your reads
	- Up to 15 read replicas
	- Within AZ, Cross AZ, Cross Region
	- Replication is Async, so reads are eventually consistent
	- replicas can be promoted to their own DB
	- Application must update the connection string to leverage read replicas
	- Use Case:
		- production DB that is taking on normal load
		- You want to run a reporting app to run some analytics
		- Create a Read Replica to run this new workload there
		- prod app is unaffected
		- Read Replicas are used for Select(=read) only kind of statements (not Insert,Update,Delete)
		- ![[Pasted image 20250721203708.png]]
	- Network cost:
		- for Replicas that are in the same region, you don't pay the network cost (it usually does when moving data from AZ to AZ)
		- ![[Pasted image 20250721203835.png]]
	- RDS Multi AZ (Disaster REcovery)
		- Sync replication
		- One DNS name - automatic app failover to standby
		- Increase availability
		- Failover in cas of loss of AZ, loss of network, instance or storage failure
		- No manual intervention in apps
		- Not used for scaling
		- Read Replicas can be setup as Multi AZ for disaster Recovery
		- ![[Pasted image 20250721204012.png]]
	- From Single AZ to Multi AZ
		- Zero downtime operation (no need to stop the DB)
		- click on modify for the database
		- following happens internally:
			- a snapshot is taken
			- a new db is restored from the snapshot in a new AZ
			- synchronization is established between the two databases
			- ![[Pasted image 20250721204143.png]]
	- RDS Custom
		- Managed Oracle and Microsoft SQL Server Database with OS and db customization
		-  RDS: Automates setup, operation,scaling of DB in AWS
		- Custom: access to the underlying database and OS so you can 
			- Configure settings
			- Install patches
			- Enable native features
			- Access  the underlying EC2 instance using SSH or SSM Session Manager
		- Deactivate Automation Mode to perform your customization, better to take a DB snapshot before
		- ![[Pasted image 20250721205301.png]]
		- RDS vs RDS Custom:
			- RDS: entire database and OS to be managed by AWS
			- RDS custom: full admin access to the underlying OS and the database (Oracle and SQL Server Database)

#### RDS backups
- Automated backups:
	- Daily full backup of the database (during the backup window)
	- Transaction logs are backed-up by RDS every 5 minutes
	- => ability to restore to any point in time (from oldest backup to 5 min utes ago)
	- 1 to 35 days of retention, set 0 to disable automated backups
- Manual DB snapshots
	- Manually triggered by the user
	- Retention of backup for as long as you want
- Trick: in a stopped RDS db, you will still pay for storage. if you plan on stopping it for a long time, you should snapshot and restore instead


#### RDS Proxy
- Fully managed database proxy for RDS
- Allows apps to pool and share DB connections established with the database
- **Improving database efficiency by reducing the stress on database resources (CPU,RAM) and minimize open connections (and timeouts)**
- Serverless, autoscaling, highly available (multi-AZ)
- Reduced RDS & Aurora failover time by up to 66%
- Supports RDS (MySQL, Postgre,MariaDB, MS SQL Server) and Aurora (MySQL,PostgreSQL)
- No code changes requried for most apps
- Enforce IAM Authentication for DB and securely store credentials in AWS Secrets manager
- Never publicly accessible (must be accessed from VPC)
- ![[Pasted image 20250721214351.png]]
- 

### Amazon Aurora
- Aurora is a proprietary technology from AWS 
- Postgres and MySQL are both supported as Aurora DB ( that means your drivers will work as if Aurora was a Postgres or MySQL database)
- is AWS cloud optimized and claims 5x performance improvement over MySQL on RDS, over 3x performance of Postgres of RDS
- Storage automatically grows in increments of 10 GB, up to 128 TB
- Can have up to 15 replicas and the replication process is faster than mySQL (sub 10ms replica lag) 
- failover is instantaneous. HA native
- costs more than RDS (20% more) - but is more efficient
- Features
	- Automatic failover
	- Backup and Recovery
	- Isolation and security
	- Industry compliance
	- Push button scaling
	- Automated Patching with Zero Downtime
	- Advanced Monitoring
	- Routine Mantenance
	- Backtrack: restore data at any point of time without using backups
#### High availability and Read Scaling
- 6 copies of your data across 3 AZ:
	- 4 copies out of 6 needed for writes
	- 3 copies out of 6 need for reads
	- self healing with peer to peer replication
	- storage is striped across 100s of volumes
- One Aurora instance takes writes (master)
- Automated failover for master in less than 30 secs
- Master + up to 15 Aurora Read Replicas serve reads
- Support for Cross Region Replication
- ![[Pasted image 20250721205816.png]]

#### Aurora DB Cluster
 ![[Pasted image 20250721205924.png]]

#### Aurora replicas - Auto scaling
 ![[Pasted image 20250721211924.png]]
#### Aurora - custom endpoints
- Defines a subset of Aurora instances as a custom endpoint
- example: run analytical queries on specific replicas
- The Reader Endpoint is generally not used after defining Custon Endpoints
- ![[Pasted image 20250721212039.png]]
- After defining custom endpoints
- ![[Pasted image 20250721212105.png]]
- Create different endpoints for subsets of Aurora instances

#### Aurora serverless
- Automated database instantiation and auto scaling based on actual usage
- Good for infrequent, intermitten or unpredictable workloads
- No capacity planning needed
- Pay per second, can be more cost-effective
- ![[Pasted image 20250721212238.png]]

#### Global Aurora
- Aurora Cross Region Read Replicas:
	- Useful for DR
	- simple to put in place
- Aurora Global Database (recommended):
	- 1 Primary Region (read/write)
	- Up to 5 secondary (read-only) regions, replication lag is less than 1 second
	- Up to 16 read replicas per secondary region
	- helps for decreasing latency
	- Promoting another region (for disaster recovery) has an RTO of <1 minute
	- Typical cross region replication take less than 1 second
	- ![[Pasted image 20250721212419.png]]

#### Aurora Machine Learning
- Enables you to add ML based predictions to your app via SQL
- simple, optimized and secure integration between Aurora and AWS ML services
- Supported services
	- Amazon SageMaker (use with any ML model)
	- Amazon Comprehend (for sentiment analysis)
- You don't need to have ML experience
- Use cases: fraud detection, ads targeting, sentiment analysis, product recommendations
- ![[Pasted image 20250721212543.png]]


#### Babelfish for Aurora PostgresQL
- Allows Aurora PostgreSQL to understand commands targeted for MS SQL Server
- Microsoft SQL server based applications can work on Aurora PostgreSQL
- Requires no to little code changes (using the same MS SQL Server Client driver)
- The same applications can be used after a migration of your database (using AWS SCT and DMS)
- ![[Pasted image 20250721212812.png]]


#### Aurora Backups
- Automated backups
	- 1 to 35 days (cannot be disabled)
	- point-in-time recovery in that timeframe
- Manual DB snapshots
	- Manually triggered by the user
	- Retention of backup for as long as you want

#### RDS & Aurora Restore options
- Restoring a RDS / Aurora backup or a snapshot creates a new database
- Restoring MySQL RDS database from S3
	- Create a backup of your on-premises database
	- Store it on Amazon S3 (object storage)
	- Restore the backup file onto a new RDS instance running MySQL
- Restoring MySQL Aurora cluster from D3
	- Create a backup of your on-premises database using Percona XtraBackup
	- Store the backup file on Amazon S3
	- Restore the backup file onto a new Aurora cluster running MySQL
#### Aurora Database Cloning
- Create a new Aurora DB cluster from an existing one
- Faster than snapshot and restore
- Uses `copy-on-write` protocol
	- Initially, the new DB cluster uses the same data volume as the original DB cluster (fast and efficient - no copying is needed)
	- When updates are made to the new DB cluster data, then additional storage is allocated and data is copied to be separated
- Very fast and cost-effective
- Useful to create a "staging" database from a "production" database without impacting the production database
- ![[Pasted image 20250721213535.png]]

#### RDS & Aurora Security
- At-rest encryption
	- Databgase master and replicas encryption using AWS KMS - must be difined as launch time
	- If the master is not encrypted, the read replicas cannot be encrypted
	- To encrypt an un-sencrypted database, go through a DB snapshot & restore as encrypted
- In-flight encryption: TLS-readyt by default, use the AWS TLS root certificates client-side
- IAM Authentication: IAM roles to connect to your database (instead of username/pw)
- Security groups: Control network access to your RDS / Aurora DB
- No SSH available except for RDS custom
- Audit logs can be enabled and sent to cloudwatch logs for longer retention




### Amazon ElastiCache
- RDS is to get managed Relational Databases
- ElastiCache is to get managed Redis or Memcached
- caches are in-memory databases with really high performance, low latency
- Helps reduce load off of databases for read intensive workloads
- Helps make your application stateless
- AWS takes care of OS maintenance / patching, optimizations, setup, configuration,monitoring,failure recovery and backups
- Using ElastiCache involves heavy application code changes

#### DB Cache
- Application queries ElastiCache, if not available, get from RDS and store in ElastiCache
- Help relive load in RDS
- Cache must have an invalidation strategy to make sure only the most current data is used in there
- ![[Pasted image 20250722232938.png]]

#### User Session Store
- User logs into any of the application
- The application writes the session data into ElastiCache
- The user hits another instance of our application 
- The instance retrieves the data and the user is already logged in 
- ![[Pasted image 20250722233026.png]]

#### Redis vs Memcached

- Redis:
	- Multi AZ with Auto-Failover
	- Read Replicas to scale reads and have high availability
	- Data Durability using AOF persistence
	- Backup and restore features
	- Supports Sets and Sorted Sets
- Memcached:
	- Multi node for partitioning of data (sharding)
	- No high availability (replication)
	- Non persistent
	- Backup and restore (serverless)
- ![[Pasted image 20250722233205.png]]

#### Cache Security
- ElastiCache supports IAM authentication for Redis
- IAM policies on ElastiCache are onjly used for AWS API-level security
- Redis AUTH
	- you can set a password/token when you create a redis cluster
	- This is an extra level of security for your cache (on top of Security groups)
	- Support SSL in flight encryption
- Memcached
	- Supports SASL - based authentication3
- ![[Pasted image 20250722233712.png]]

#### Patterns for ElastiCache
- Lazy Loading: all the read data is cached, data can become stale in cache
	- ![[Pasted image 20250722233828.png]]
- Write Through: Adds or update data in the cache when written to a DB (no stale data)
- Session Store: store temporary session data in a cache  (using TTL features)

#### Redis Use Case
- Gaming leaderboards are computationally complex
- **Redis sorted sets** guarantee both uniqueness and element ordering
- Each time a new element added, it's ranked in real time, then added in the correct order
- ![[Pasted image 20250722233923.png]]


#### List of Ports to be familiar with

Here's a list of **standard** ports you should see at least once. You shouldn't remember them (the exam will not test you on that), but **you should be able to differentiate between an Important (HTTPS - port 443) and a database port (PostgreSQL - port 5432)** 

**Important ports:**

- FTP: 21
    
- SSH: 22
    
- SFTP: 22 (same as SSH)
    
- HTTP: 80
    
- HTTPS: 443
    

**vs RDS Databases ports:**

- PostgreSQL: 5432
    
- MySQL: 3306
    
- Oracle RDS: 1521
    
- MSSQL Server: 1433
    
- MariaDB: 3306 (same as MySQL)
    
- Aurora: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)
    

  

> Don't stress out on remember those, just read that list once today and once before going into the exam and you should be all set :)
> 
> Remember, you should just be able to differentiate an "Important Port" vs an "RDS database Port".