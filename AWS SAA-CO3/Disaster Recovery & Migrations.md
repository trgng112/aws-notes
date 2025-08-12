 ### Disaster Recovery Overview
- Any event that has a negative impact on a company's business continuity or finances is a disaster
- Disaster recovery (DR) is about preparing for and recovering from a disaster
- what kind of disaster recovery?
	- on-premise => on-premise: traditional DR, and very expensive
	- On-premise => AWS Cloud: hybrid recovery
	- AWS Cloud Region A => AWS Cloud Region B
- Need to define two terms:
	- RPO: Recovery Point Objective
	- RTO: Recovery Time Objective

### RPO & RTO
- ![[Pasted image 20250811174120.png]]

### DR Strategies
- Backup and Restore
- Pilot Light
- Warm Standby
- Hot Site / Multi Site Approach
- ![[Pasted image 20250811174152.png]]

#### Backup and restore (High RPO)
![[Pasted image 20250811174242.png]]
#### Pilot Light
- A Small version of the app is always running in the cloud
- Useful for the critical core (pilot light)
- Very similar to Backup and Restore
- Faster than Backup and Restore as critical systems are already up
- ![[Pasted image 20250811174350.png]]

#### Warm Standby
- Full system is up and running, but at minimum size
- Upon disaster, we can scale to production load
- ![[Pasted image 20250811174418.png]]

#### Multi Site/ Hot Site Approach
- Very low RTO (minutes or seconds) - very expensive
- Full production scale is running AWS and On Premise
- ![[Pasted image 20250811174503.png]]

#### All AWS Multi Region
- ![[Pasted image 20250811174524.png]]

### DR Tips
- Backups:
	- EBS Snapshots, RDS automated backups/ Snapshots,etc...
	- Regular pushes to S3/ S3 IA / Glacier, Lifecycle Policy, Cross Region Replication
	- From on-premise: snowball or storage gateway
- High Availability
	- Use Route53 to migrate DNS over from Region to REgion
	- RDS Mutli-AZ, ElastiCache Multi-AZ, EFS,S3
	- Site to Site VPN as a recovery from Direct Connect
- Replication
	- RDS Replication (Cross Region), AWS Aurora + Global databases
	- Database replication from on premise to RDS
	- Storage Gateway
- Automation
	- CloudFormation / Elastic Beanstalk to re-create a whole new environment
	- Recover/Reboot EC2 instances with CW if alarms fail
	- AWS Lambda functions for customized automations
- Chaos
	- Netflix has a 'simian army' randomly terminating EC2


## Database Migration Service (DMS)
 - Quickly and securely migrate dbs to AWS, resilient, self healing
 - The source db remains available during the migration
 - Supports:
	 - Homogeneous migrations: ex Oracle to Oracle
	 - Heterogeneous migrations: ex Microsoft SQL Server to Aurora
 - Continuous Data Replication using CDC
 - You must create an EC2 instance to perform the replication tasks
 - ![[Pasted image 20250811174931.png]]
### DMS Sources and Targets
- Sources:
	- On premises and EC2 instances dbs: Oracle, MS SQL Server,MySQL,MariaDB,PostGresql
	- Azure: Azure SQL DB
	- Amazon RDS: all including Aurora
	- Amazon S3
	- DocumentDB
- Targets:
	- On Premise and EC2 instances dbs: Oracle, MSSQL Server, MySQL,MariaDB,PostgreSQL,SAP
	- RDS
	- Redshift , dynamoDB, S3
	- OpenSearch Service
	- Kenesis Data Streams
	- Kafka
	- DocumentDB & Amazon Neptune
	- Redis & Bablefish

### AWS Schema Conversion Tool (SCT)
- Convert your DB's schema from one engine to another
- Example: OLTP (SQL Server or Oracle) to MySQL, PostgreSQL,Aurora,
- Example OLAP (Teradata or Oracle) to Amazon Redshift
- **You do not need to use SCT if you are migrating the same DB engine** 
	- EX: on premise postgre => RDS Postgre
	- DB engine is still Postgre (RDS is the platform)
	- ![[Pasted image 20250811175256.png]]

### DMS - Continuous Replication
![[Pasted image 20250811175343.png]]

## RDS & Aurora MySQL Migrations
- RDS MySQL to Aurora MySQL
	- Option 1 : DB Snapshots from RDS MySQL restored as MySQL Aurora DB
	- Option 2 : Create an Aurora Read Replica from your RDS MySQL and when the replication lag is 0, promote it as its own DB cluster (can take time and cost $)
- External MYSQL to Aurora MySQL
	- Option1:
		- Use Percona XtraBackup to create a file backup in S3
		- Create an Aurora MySQL DB from S3
	- Option 2 :
		- Create an Aurora MySQL DB
		- Use the mysqldump utility to migrate MySQL into urora (slower than S3 method)
- Use DMS if both dbs are up and running 
- ![[Pasted image 20250811175942.png]]

### RDS & Aurora PostgreSQL migrations
- RDS Posgre to Aurora Postgre
	- Same as MySQl
- External Postgre to Aurora Postgre
	- Create a backup and put it in S3
	- Import it using the aws_s3 Aurora extension
- Use DMS if both dbs are up and running
- ![[Pasted image 20250811180107.png]]


## On - Premise strategy with AWS
- Ability to download Amazon Linux 2 AMI as a VM (.iso format)
	- VMWare, KVM,VirtualBox(oracleVM),Microsoft HyperV
- VM Import/ Export
	- Migrate existing applications into EC2
	- Create a DR repository strategy for your on-premise VMs
	- Can export back the VMs from EC2 to on-premise
- AWS Application Discovery Service
	- Gather information about your on-premise servers to plan a mgiration
	- Server utilization and dependency mappings
	- Track with AWS Migration Hub
- AWS Database Migration Service (DMS)
	- replicate on-premise => AWS, AWS => AWS, AWS => On premise
	- works with various db techs (Oracle , MYSQL,DynamoDB,etc,....)
- AWS Server Migration Service( SMS)
	- Incremental replication of on-premise live servers to AWS

## AWS Backup
- Fully managed service
- Centrally manage and automate backups across AWS services
- No need to create custom scripts and manual processes
- Supported Services:
	- EC2/EBS
	- S3
	- RDS (all DB engines)/ Aurora/ DynamoDB
	- DocumentDB/Neptune
	- EFS,FSx(Listre & Windows File Server)
	- AWS Storage Gateway (volume gateway)
- Supports cross-region backups
- Supports cross-account backups
- Supports PITR for supported services
- On-Demand and Scheduled backups
- Tag-based backup policies
- You create backup policies known as **Backup Plans**
	-  Backup frequency (every 12 hrs, daily, weekly, monthly, cron expression)
	- Backup window
	- Transition to Cold Storage (Never, Days, Weeks, Months, Years)
	- Retention Period (Always, Days, Weeks, Months, Years)
- ![[Pasted image 20250811180627.png]]

### AWS Backup Vault Lock
- Enforce a WORM (Write Once Read Many) state for all the backups that you store in your AWS Backup Vault
- Additional layer of defense to protect your backup against:
	- Inadvertent or malicious delete operations
	- Updates that shorten or alter retention periods
- Even the root user cannot delete backups when enabled
- ![[Pasted image 20250811180732.png]]

## Application Migration Service (MGN)
- Plan migration projects by gathering information about on-premises data centers
- Server utilization data and dependency mapping are important for migrations
- **Agentless Discovery (AWS agentless Discovery Connector)**
	- VM inventory, configs, and performance history such as CPU ,memory and disk usage
- **Agent - based Discovery (AWS Application Discovery Agent)**
	- System configs, system performance, running processes, and details of the network connections between systems
- Resulting data can be viewed within AWS Migration Hub
- Lift - and - shift (rehost) solution which simplify migratin applications to AWS
- converts your physical, virtual, and cloud based servers to run natively on AWS
- Minimal downtime,reduced costs,
- ![[Pasted image 20250811181349.png]]
## Transferring large amount of data into AWS
Example: transfer 200TB of data in the cloud. We have a 100Mbps internet connection.
- **Over the internet/site-to-site VPN**:
	- Immediate to setup
	- Will take 200TB x 1000 GB x 1000MB x 8MB / 100 Mbps = 16,000,000s = 185d
- **Over direct connect 1Gbps**:
	- Long for the one-time setup (over a month)
	- Will take 200Tb x 1000GB x 8gb / 1 Gbps = 1,600,00s = 18.5d
- **Over Snowball**:
	- Takes about 1 week for the end to end transfer
	- can be combined with DMS
- **For on-going replication/transfers:** -Site to Site VPN or DX with DMS or DataSync
## VMWare Cloud on AWS
- Some customers use VMware Cloud to manage their on-premises Data Center
- They want to extend the Data Center capacity to AWS, but keep using the VMware Cloud software
- ... Enter VMware Cloud on AWS
- Use cases:
	- Migrate your VMware vSphere-based workloads to AWS
	- Run your production workload across VMware vSphere-based private,public and hybrid cloud environments
	- Have a DR strategy
- ![[Pasted image 20250811181821.png]]