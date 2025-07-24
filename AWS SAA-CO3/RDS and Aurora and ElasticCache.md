# RDS, Aurora, and ElastiCache

## RDS - Amazon Relational Database Service

### Overview
Amazon Relational Database Service (RDS) is a managed database service that uses SQL as a query language, allowing you to create databases that are managed by AWS.

### Supported Database Engines
- **PostgreSQL**
- **MySQL**
- **MariaDB**
- **Oracle**
- **SQL Server**
- **Amazon Aurora** (AWS Proprietary database)

### RDS Managed Service Features
- **Automated provisioning** and OS patching
- **Continuous backups** and restore to specific timestamps (Point In Time Restore)
- **Monitoring dashboards**
- **Read replicas** for improved read performance
- **Multi AZ setup** for Disaster Recovery
- **Maintenance windows** for upgrades
- **Scaling capability** (vertical and horizontal)
- **Storage backed by EBS**

**Note**: You cannot SSH into your underlying RDS instances

### Storage Auto Scaling
- **Purpose**: Helps you increase storage on your RDS DB instance dynamically
- **Functionality**: Scales automatically when detecting you are running out of free database storage
- **Benefits**: Avoid manually scaling your database storage
- **Configuration**: You must set Maximum Storage Threshold (maximum limit for DB storage)

#### Auto Scaling Triggers
Automatically modifies storage when:
- Free storage is less than 10% of allocated storage
- Low storage lasts at least 5 minutes
- 6 hours have passed since last modification

**Use Cases**: Applications with unpredictable workloads
**Compatibility**: Supports all RDS database engines

![[Pasted image 20250721203412.png]]

### RDS Read Replicas

#### Overview
- **Purpose**: Helps you scale your reads
- **Capacity**: Up to 15 read replicas
- **Deployment**: Within AZ, Cross AZ, Cross Region
- **Replication**: Asynchronous, so reads are eventually consistent
- **Promotion**: Replicas can be promoted to their own database
- **Application Changes**: Must update connection string to leverage read replicas

#### Use Case Example
- Production database taking normal load
- Need to run reporting application for analytics
- Create Read Replica to run new workload
- Production application remains unaffected
- Read Replicas used for SELECT (read) only statements (not INSERT, UPDATE, DELETE)

![[Pasted image 20250721203708.png]]

#### Network Costs
- **Same Region**: No network cost for replicas in the same region
- **Cross Region**: Standard data transfer charges apply

![[Pasted image 20250721203835.png]]

### RDS Multi-AZ (Disaster Recovery)

#### Overview
- **Replication**: Synchronous replication
- **DNS**: One DNS name with automatic application failover to standby
- **Purpose**: Increase availability
- **Failover Scenarios**: Loss of AZ, network, instance, or storage failure
- **Manual Intervention**: None required in applications
- **Scaling**: Not used for scaling (use Read Replicas instead)

**Note**: Read Replicas can be setup as Multi-AZ for disaster recovery

![[Pasted image 20250721204012.png]]

#### Migration from Single AZ to Multi AZ
- **Downtime**: Zero downtime operation (no need to stop the database)
- **Process**: Click "modify" for the database

**Internal Process**:
1. A snapshot is taken
2. New database restored from snapshot in new AZ
3. Synchronization established between the two databases

![[Pasted image 20250721204143.png]]

### RDS Custom

#### Overview
- **Support**: Managed Oracle and Microsoft SQL Server Database with OS and database customization
- **Standard RDS**: Automates setup, operation, and scaling of database in AWS
- **RDS Custom**: Access to underlying database and OS

#### Custom Capabilities
- Configure settings
- Install patches
- Enable native features
- Access underlying EC2 instance using SSH or SSM Session Manager

**Important**: Deactivate Automation Mode to perform customization. Recommended to take DB snapshot before customization.

![[Pasted image 20250721205301.png]]

#### RDS vs RDS Custom Comparison
- **RDS**: Entire database and OS managed by AWS
- **RDS Custom**: Full admin access to underlying OS and database (Oracle and SQL Server only)

### RDS Backups

#### Automated Backups
- **Frequency**: Daily full backup during backup window
- **Transaction Logs**: Backed up every 5 minutes
- **Point-in-time Recovery**: From oldest backup to 5 minutes ago
- **Retention**: 1 to 35 days (set 0 to disable automated backups)

#### Manual DB Snapshots
- **Trigger**: Manually triggered by user
- **Retention**: As long as you want

**Cost Tip**: Stopped RDS database still incurs storage costs. For long-term stops, snapshot and restore instead.

### RDS Proxy

#### Overview
- **Purpose**: Fully managed database proxy for RDS
- **Function**: Allows applications to pool and share database connections
- **Benefits**: Improves database efficiency by reducing stress on database resources (CPU, RAM) and minimizing open connections and timeouts

#### Features
- **Serverless**: Autoscaling, highly available (multi-AZ)
- **Failover**: Reduces RDS & Aurora failover time by up to 66%
- **Compatibility**: Supports RDS (MySQL, PostgreSQL, MariaDB, MS SQL Server) and Aurora (MySQL, PostgreSQL)
- **Code Changes**: No code changes required for most applications
- **Security**: Enforces IAM Authentication and securely stores credentials in AWS Secrets Manager
- **Access**: Never publicly accessible (must be accessed from VPC)

![[Pasted image 20250721214351.png]]

## Amazon Aurora

### Overview
- **Technology**: Proprietary technology from AWS
- **Compatibility**: PostgreSQL and MySQL supported (drivers work as if Aurora was PostgreSQL or MySQL)
- **Performance**: 5x performance improvement over MySQL on RDS, 3x performance over PostgreSQL on RDS
- **Storage**: Automatically grows in increments of 10GB, up to 128TB
- **Replicas**: Up to 15 replicas with faster replication than MySQL (sub 10ms replica lag)
- **Availability**: Instantaneous failover, high availability native
- **Cost**: 20% more than RDS but more efficient

### Aurora Features
- **Automatic failover**
- **Backup and Recovery**
- **Isolation and security**
- **Industry compliance**
- **Push button scaling**
- **Automated Patching** with Zero Downtime
- **Advanced Monitoring**
- **Routine Maintenance**
- **Backtrack**: Restore data at any point in time without using backups

### High Availability and Read Scaling

#### Data Replication
- **Copies**: 6 copies of data across 3 AZ
- **Write Requirements**: 4 copies out of 6 needed for writes
- **Read Requirements**: 3 copies out of 6 needed for reads
- **Self-healing**: Peer-to-peer replication
- **Storage**: Striped across hundreds of volumes

#### Instance Configuration
- **Master**: One Aurora instance handles writes
- **Failover**: Automated failover for master in less than 30 seconds
- **Read Replicas**: Master + up to 15 Aurora Read Replicas serve reads
- **Cross Region**: Supports Cross Region Replication

![[Pasted image 20250721205816.png]]

### Aurora DB Cluster

![[Pasted image 20250721205924.png]]

### Aurora Replicas - Auto Scaling

![[Pasted image 20250721211924.png]]

### Aurora Custom Endpoints

#### Overview
- **Purpose**: Define subset of Aurora instances as custom endpoint
- **Use Case**: Run analytical queries on specific replicas
- **Note**: Reader Endpoint generally not used after defining Custom Endpoints

![[Pasted image 20250721212039.png]]

#### After Defining Custom Endpoints

![[Pasted image 20250721212105.png]]

Create different endpoints for subsets of Aurora instances for specific workloads.

### Aurora Serverless

#### Overview
- **Function**: Automated database instantiation and auto-scaling based on actual usage
- **Best For**: Infrequent, intermittent, or unpredictable workloads
- **Capacity Planning**: None needed
- **Pricing**: Pay per second, can be more cost-effective

![[Pasted image 20250721212238.png]]

### Global Aurora

#### Aurora Cross Region Read Replicas
- **Purpose**: Useful for disaster recovery
- **Setup**: Simple to implement

#### Aurora Global Database (Recommended)
- **Primary Region**: 1 region for read/write
- **Secondary Regions**: Up to 5 read-only regions
- **Replication Lag**: Less than 1 second
- **Read Replicas**: Up to 16 per secondary region
- **Benefits**: Decreases latency
- **Disaster Recovery**: Promoting another region has RTO of <1 minute
- **Cross Region Replication**: Typically less than 1 second

![[Pasted image 20250721212419.png]]

### Aurora Machine Learning

#### Overview
- **Purpose**: Add ML-based predictions to applications via SQL
- **Integration**: Simple, optimized, and secure integration between Aurora and AWS ML services

#### Supported Services
- **Amazon SageMaker**: Use with any ML model
- **Amazon Comprehend**: For sentiment analysis

#### Benefits
- **Experience**: No ML experience required
- **Use Cases**: Fraud detection, ads targeting, sentiment analysis, product recommendations

![[Pasted image 20250721212543.png]]

### Babelfish for Aurora PostgreSQL

#### Overview
- **Purpose**: Allows Aurora PostgreSQL to understand commands targeted for MS SQL Server
- **Compatibility**: Microsoft SQL Server applications can work on Aurora PostgreSQL
- **Code Changes**: Requires minimal code changes
- **Migration**: Same applications can be used after database migration (using AWS SCT and DMS)

![[Pasted image 20250721212812.png]]

### Aurora Backups

#### Automated Backups
- **Retention**: 1 to 35 days (cannot be disabled)
- **Recovery**: Point-in-time recovery within timeframe

#### Manual DB Snapshots
- **Trigger**: Manually triggered by user
- **Retention**: As long as desired

### RDS & Aurora Restore Options

#### General Restore Process
- **New Database**: Restoring RDS/Aurora backup or snapshot creates new database

#### Restoring MySQL RDS from S3
1. Create backup of on-premises database
2. Store on Amazon S3
3. Restore backup file onto new RDS instance running MySQL

#### Restoring MySQL Aurora from S3
1. Create backup using Percona XtraBackup
2. Store backup file on Amazon S3
3. Restore backup file onto new Aurora cluster running MySQL

### Aurora Database Cloning

#### Overview
- **Purpose**: Create new Aurora DB cluster from existing one
- **Speed**: Faster than snapshot and restore
- **Protocol**: Uses copy-on-write protocol

#### How It Works
1. **Initial State**: New DB cluster uses same data volume as original (fast, no copying needed)
2. **Updates**: When updates made to new cluster, additional storage allocated and data copied
3. **Benefits**: Very fast and cost-effective

**Use Case**: Create "staging" database from "production" database without impacting production

![[Pasted image 20250721213535.png]]

### RDS & Aurora Security

#### At-Rest Encryption
- **Method**: Database master and replicas encryption using AWS KMS
- **Timing**: Must be defined at launch time
- **Limitation**: If master not encrypted, read replicas cannot be encrypted
- **Workaround**: To encrypt un-encrypted database, go through DB snapshot & restore as encrypted

#### In-Flight Encryption
- **Default**: TLS-ready by default
- **Implementation**: Use AWS TLS root certificates client-side

#### Access Control
- **IAM Authentication**: IAM roles to connect to database (instead of username/password)
- **Network Security**: Security groups control network access to RDS/Aurora DB
- **SSH Access**: Not available except for RDS Custom
- **Audit Logs**: Can be enabled and sent to CloudWatch Logs for longer retention

## Amazon ElastiCache

### Overview
- **Purpose**: RDS is for managed Relational Databases; ElastiCache is for managed Redis or Memcached
- **Technology**: In-memory databases with high performance and low latency
- **Benefits**: Reduces load on databases for read-intensive workloads and makes applications stateless
- **Management**: AWS handles OS maintenance, patching, optimizations, setup, configuration, monitoring, failure recovery, and backups

**Note**: Using ElastiCache involves heavy application code changes

### ElastiCache Use Cases

#### Database Cache
1. Application queries ElastiCache
2. If data not available, get from RDS and store in ElastiCache
3. Helps relieve load on RDS
4. Cache must have invalidation strategy for current data

![[Pasted image 20250722232938.png]]

#### User Session Store
1. User logs into any application instance
2. Application writes session data into ElastiCache
3. User hits another application instance
4. Instance retrieves data and user remains logged in

![[Pasted image 20250722233026.png]]

### Redis vs Memcached

#### Redis Features
- **Multi-AZ** with Auto-Failover
- **Read Replicas** to scale reads and provide high availability
- **Data Durability** using AOF persistence
- **Backup and restore** features
- **Data Structures**: Supports Sets and Sorted Sets

#### Memcached Features
- **Multi-node** for partitioning of data (sharding)
- **No high availability** (replication)
- **Non-persistent**
- **No backup and restore**
- **Serverless**

![[Pasted image 20250722233205.png]]

### Cache Security

#### Redis Security
- **IAM Authentication**: ElastiCache supports IAM authentication for Redis
- **API Security**: IAM policies only used for AWS API-level security
- **Redis AUTH**: Set password/token when creating Redis cluster for extra security
- **Encryption**: Supports SSL in-flight encryption

#### Memcached Security
- **Authentication**: Supports SASL-based authentication

![[Pasted image 20250722233712.png]]

### ElastiCache Patterns

#### Lazy Loading
- **Function**: All read data is cached
- **Limitation**: Data can become stale in cache

![[Pasted image 20250722233828.png]]

#### Write Through
- **Function**: Adds or updates data in cache when written to database
- **Benefit**: No stale data

#### Session Store
- **Function**: Store temporary session data in cache
- **Feature**: Uses TTL (Time To Live) features

### Redis Use Case: Gaming Leaderboards

#### Overview
- **Challenge**: Gaming leaderboards are computationally complex
- **Solution**: Redis Sorted Sets guarantee both uniqueness and element ordering
- **Real-time Ranking**: Each new element added is ranked in real time and added in correct order

![[Pasted image 20250722233923.png]]

## Important Ports Reference

### Standard Ports
**Important Ports:**
- **FTP**: 21
- **SSH**: 22
- **SFTP**: 22 (same as SSH)
- **HTTP**: 80
- **HTTPS**: 443

### RDS Database Ports
- **PostgreSQL**: 5432
- **MySQL**: 3306
- **Oracle RDS**: 1521
- **MSSQL Server**: 1433
- **MariaDB**: 3306 (same as MySQL)
- **Aurora**: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)

**Exam Tip**: You should be able to differentiate between "Important Ports" vs "RDS Database Ports" rather than memorizing specific numbers.




