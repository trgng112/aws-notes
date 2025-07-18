Elastic Compute Cloud = Infrastructure as a service 
	- Renting virtual machines
	- Storing data on virtual drives
	- Distributing load across machines
	- scale the services with auto scales

Can choose
	- OS
	- Cpu 
	- Ram
	- Storage space
		- Network attached EBS EFS
		- hardware EC2 instance store
	- Network card
		- speed and ip address
	- Firewall rules
	- Bootstrap scripts: EC2 user data


### EC2 types

m5.2xlarge
- m:instance class
- 5: generation
- 2xlarge: size within the instance class

General purpose

- Great for diversity, balance between compute memory networking
- t3.micro

Compute Optimized
- compute intensive task
- batch processing workloads, media, web servers, game servers, ML

Memory optimized
- process large data sets in memory
- High performance dbs
- web cache
- in memory db for business intelligence
- real time processing of big unstructured data

Storage optimized
- high, sequential read and write access to large data sets on local storage
- Relational and noSQL dbs
- cache for in memory db (Redis)
- high frequency transaction
- data warehouse

![[Pasted image 20250716210405.png]]

### Security groups

- Security groups are fundamental of network security
- Are firewalls
- Control how traffic is allowed into or out of EC2
- Can contain allow rules
- can reference by IP or by security group
- ![[Pasted image 20250716210531.png]]
- Regulates
	- Access to ports
	- Authorised IP ranges IPv4 and IPv6
	- Control of inbound network
	- Control of outbound network
	- ![[Pasted image 20250716210659.png]]
- Can be attached to multiple instances
- Region / VPC scoped
- live outside of EC2
- ![[Pasted image 20250716210853.png]]
- Classic ports
	- 22 = SSH  - log into linux instance
	- 21 = FTP - upload file
	- 22 = SFTP - upload files using ssh
	- 80 = HTTP - access websites
	- 443 = HTTPS - access secured
	- 3389 = RDP (remote desktop protocol) - log into a windows instance
-

### EC2 instances purchasing options
- ![[Pasted image 20250716213146.png]]
- On-Demand instances : short workload, pay by second
	- Linux or Windows: billing per secs
	- Other OS billing per hour
	- Highest cost but no upfront payment
	- no longterm commitment
	- recommended for short term and uninterrupted
- Reserved (1 & 3 years)
	- Reserved instances - long workloads
	- Convertible Reserved Instances - long workloads with flexible instances
	- 72% discount upto
	- reserve specific instance attribute (InstanceType, Region,Tenancy,OS)
	- payment options
	- reserved instance scope: regional or Zonal
	- buy and sell in reserved instance marketplace
	- Convertible Reserved Instance: can change the instance type, instance family,os,scope,tenancy,
	- up to 66% discount
- Saving Plans (1 - 3 years): commitment to an amount of usage, long hours
	- up to 72%
	- commit to a certain type of usage (10$ /hour for 1 year)
	- usage beyond will billed as on demand
	- locked to specific instance family and aws region
- Spot instances: short workload, cheap, can lose instances
	- up to 90% discount
	- instances that you can lose at any point if your max price is less than the current spot price
	- most cost efficient
	- usage for workloads that are resilient to failure - Batch jobs, data analysis, image processing, distributed workloads
	- ![[Pasted image 20250716214137.png]]
	- ![[Pasted image 20250716214105.png]]
- Dedicated host: entire physical server
	- Ec2 instance capacity fully dedicated
	- address compliance requirements, use server bound software license
- Dedicated instances: no customer will share
- Capacity reservations: reserve capacity in specific AZ for any duration
	- Reserve on demand capacity in specific AZ for any duration
	- No time commitment
	- combine with Regional reserved instances and saving plans to benefit from billing discounts
	- charge at on demand rate whether you run instances or not


### Private vs Public IP PIv4
- IPv4 is the most common format
- IPv6 is newer and solves problesm in IoT
- v4 allows for 3.7 bil different address in the public space
- v4: [0-255].[0-255].[0-255].[0-255]


Public
	Can be identified on WWW
	Must be unique
	 Can be geo-located
Private 
- can be indentified on private network
- must be unique
- 2 different networks can have the same private ip
- connect to WWW using NAT + internet gateway

Elastic IP
- when you stop and start an ec2 instance, it change its public IP
- if you need to have a fixed public IP for your instance, you need an elastic IP
- elastic IP is a public v4 as long as you dont delete it
- can only attach to 1 instance at a time
- can only have 5 elastic IP

### Placement groups
- Control over the ec2 instance placement strategy - can be defined by using placement groups
- Cluster: clusters instances into a low latency group in a single AZ
	- pros: great network
	- cons: if the AZ fail, all instance fails
	- use for: big data job, apps that needs extremely low latency and high network output
	- ![[Pasted image 20250717094855.png]]
- Spread: spreads instances across underlying hardware (max 7 instances per group per AZ) - critical applications
	- pros:  can span across multiple AZ, reduce risk of AZ failure, instances are on different physical hardware
	- cons: limited to 7 instances per AZ per placement group
	- use for: apps that need to maximize high availability, critical apps where each instance must be isolated from failure from each other
- Partition: spreads instances across many different partitions within an AZ. Scales to 100s of ec2 instances per group
	- pros: update 8 partitions per AZ, up to 100s ec2 instances, span multiple AZs, each partition is protected from failure, gain access to partition information as metadata
	- use case: HDFS,HBase, Cassandra, Kafka

### Elastic Network Interfaces ENI

- Logical component in a VPC that represents a virtual network card
- have the following attributes
	- Primary private ipv4, one or more secondary ipv4
	- one elastic ipv4 per private ipv4
	- one public ipv4
	- one or more security groups
	- a MAC address
- Can be created independently and attach the on the fly on ec2 instances for failovers
- AZ scoped
![[Pasted image 20250717095620.png]]

### EC2 hibernate

- RAM is reserved
- instance boot is much faster
- under the hood: RAM state is written to a file in the root EBS volume
- root EBS volume must be encrypted
- support alot of instance families
- hibernate no more than 60 days
use case: 
- long running process
- saving ram state
- services that takes time to init


### EC2 Instance Storage Section
 - EBS - Elastic Block Store Volume is a network drive you can attach to your instances while they run
 - Allow your instances to persist data, even after termination
 - Can only mounted to one instance at a time
 - AZ scoped ( cannot bound to instance in another AZ)
 - Free tier: 30gb ssd
 - similarly to a network drive 
 - can create snapshot to move across
	 - backup at a point in time
	 - not necessary to detach volume to do snapshot
	 - can copy snapshot across AZ or Region![[Pasted image 20250718221418.png]]
	 - EBS Snapshot Archive - move to archive tier - 75% cheaper - take 24 to 72 hrs to restore
	 - Recycle bin for EBS snapshot - setup rules to retain deleted snapshot so you can recover them after an accidental deletion
		 - from 1 day to 1 year retention
	 - Fast snapshot Restore FSR - Force full init of snapshot to have no latency on first use - most expensive
 - ![[Pasted image 20250718220700.png]]
 - By default, root EBS volume is deleted (if enabled)
	 - other attached EBS volume is not deleted (attribute disabled)
	 - Can be controlled in AWS console / CLI
	 - preserve root volume when instance is terminated 


### AMI - Amazon Machine Image
- customization of EC2 instance
	- your own software, configs, OS , monitoring
	- faster boot / config time because software is prepackaged
- AMI are built for a specific region - can be copied
- can launch EC2 instance from: public ami, your own ami, aws marketplace ami

Process: 
- start an EC2 instance and customize it
- stop the instance
- build an AMI - also create EBS snapshot
- launch instances from other AMIs
- ![[Pasted image 20250718222044.png]]

### EC2 Instance Store
- use for high-performance hardware disk
- better I/O performance
- lose their storage if they're stopped (ephemeral)
- Good for buffer/cache/scratch data
- risk of data loss if hardware fails
- backup and replication are your responsibility

### EBS volume types
- 6 types
	- gp2/gp3 - SSD : general purpose SSD balance price and performance
		- cost effective, low latency
		- system boot volumes, Virtual desktops, development
		- 1gb-16tib
		- gp3: baseline of 3000 iops (max 16000) and throughput of 125MiB/s (max 1000MiB/s)
		- gp2: small gp2 volumes can burst iops to 3000, size of the volume and iops are linked, max is 16000, 3 iops per GB, max at 5334gb 
	- io1/io2 block express - SSD : high performance SSD 
		- Critical business that need sustained iops perf
		- apps that need more than 16000 iops
		- great for db workloads
		- has **multi-attach** - attached the same EBS volume to multiple EC2 in the same AZ, each instance has full read and write permissions 
			- up to 16 ec2 instances at a time
			- ![[Pasted image 20250718223359.png]]
		- io1 (4 GiB - 16 TiB):
			• Max PIOPS: 64,000 for Nitro EC2 instances & 32,000 for other
			• Can increase PIOPS independently from storage size
		• io2 Block Express (4 GiB – 64 TiB):
			• Sub-millisecond latency
			• Max PIOPS: 256,000 with an IOPS:GiB ratio of 1,000:1
			• Supports EBS Multi-attach
		

	- st1 - HDD : low cost hdd for frequently accessed workloads
	- st2 - HDD: lowest cost hdd for less frequently accessed workloads
		- cannot be a boot volume
		- 125gb to 16tib
		- throughput Optimized HDD (st1)
			• Big Data, Data Warehouses, Log Processing
			• Max throughput 500 MiB/s – max IOPS 500
		 Cold HDD (sc1):
			• For data that is infrequently accessed
			• Scenarios where lowest cost is important
			• Max throughput 250 MiB/s – max IOPS 250
	- 
- characterized in size | throughput|iops
- only gp2/gp3 and io1/io2 Block Express can be used as boot volumes

### EBS encryption
- when you create an encrypted EBS volume, you get the following
	- Data at rest is encrypted inside the volume
	- All the data in flight moving between the instance and the volume is encrypted
	- All snapshots are encrypted
	- All volumes created from the snapshot
- encryption and decryption are handled transparently
- has a minimal impact on latency
- leverages key from KMS (AES-256)
- Copying an unencrypted snapshot allow encryption
- snapshots of encrypted volumes are encrypted
- How to encrypt an unencrypted EBS volume
	- Create an EBS snapshot of that volume
	- Encrypt the snapshot - copy it 
	- create new ebs volume from the snapshot - it will also be encrypted
	- attached the encrypted volume to the instance

