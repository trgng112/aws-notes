# EC2 - Elastic Compute Cloud

## Overview
Elastic Compute Cloud = Infrastructure as a Service (IaaS)
- Renting virtual machines
- Storing data on virtual drives  
- Distributing load across machines
- Scale the services with auto scaling

## EC2 Configuration Options

You can choose:
- **Operating System** (OS)
- **CPU** 
- **RAM**
- **Storage space**
  - Network attached: EBS, EFS
  - Hardware: EC2 instance store
- **Network card**
  - Speed and IP address
- **Firewall rules** (Security Groups)
- **Bootstrap scripts**: EC2 user data

## EC2 Instance Types

### Instance Naming Convention
Example: `m5.2xlarge`
- `m`: instance class
- `5`: generation  
- `2xlarge`: size within the instance class

### Instance Categories

#### General Purpose
- Great for diversity, balance between compute, memory, and networking
- Example: `t3.micro`

#### Compute Optimized
- For compute-intensive tasks
- Use cases: batch processing workloads, media transcoding, web servers, game servers, machine learning

#### Memory Optimized  
- Process large data sets in memory
- Use cases: High performance databases, web cache, in-memory databases for business intelligence, real-time processing of big unstructured data

#### Storage Optimized
- High, sequential read and write access to large data sets on local storage
- Use cases: Relational and NoSQL databases, cache for in-memory databases (Redis), high frequency transactions, data warehouses

![[Pasted image 20250716210405.png]]

## Security Groups

Security groups are fundamental components of network security in AWS:

- **Function**: Act as virtual firewalls
- **Purpose**: Control how traffic is allowed into or out of EC2 instances
- **Rules**: Can contain only allow rules (no deny rules)
- **References**: Can reference by IP address or by security group

![[Pasted image 20250716210531.png]]

### What Security Groups Regulate
- Access to ports
- Authorized IP ranges (IPv4 and IPv6)
- Control of inbound network traffic
- Control of outbound network traffic

![[Pasted image 20250716210659.png]]

### Key Characteristics
- Can be attached to multiple instances
- Scoped to Region/VPC
- Live outside of EC2 instances

![[Pasted image 20250716210853.png]]

### Common Ports
- **22** = SSH - log into Linux instance
- **21** = FTP - upload files
- **22** = SFTP - upload files using SSH
- **80** = HTTP - access websites  
- **443** = HTTPS - access secured websites
- **3389** = RDP (Remote Desktop Protocol) - log into Windows instance

## EC2 Purchasing Options

![[Pasted image 20250716213146.png]]

### On-Demand Instances
**Best for**: Short workloads, pay by second
- **Billing**: Linux/Windows per second, other OS per hour
- **Cost**: Highest cost but no upfront payment
- **Commitment**: No long-term commitment
- **Recommended for**: Short-term and uninterrupted workloads

### Reserved Instances (1 & 3 years)
**Best for**: Long workloads with predictable usage

#### Standard Reserved Instances
- Up to 72% discount
- Reserve specific instance attributes (Instance Type, Region, Tenancy, OS)
- Payment options available
- Scope: Regional or Zonal
- Can buy and sell in Reserved Instance Marketplace

#### Convertible Reserved Instances  
- Up to 66% discount
- Can change: instance type, instance family, OS, scope, tenancy
- More flexibility than standard reserved instances

### Savings Plans (1-3 years)
**Best for**: Commitment to amount of usage over long periods
- Up to 72% discount
- Commit to certain usage amount ($10/hour for 1 year)
- Usage beyond commitment billed at On-Demand rates
- Locked to specific instance family and AWS region

### Spot Instances
**Best for**: Short workloads, cost-sensitive, fault-tolerant applications
- Up to 90% discount
- Can lose instances if your max price < current spot price
- Most cost-efficient option
- **Use cases**: Batch jobs, data analysis, image processing, distributed workloads

![[Pasted image 20250716214137.png]]
![[Pasted image 20250716214105.png]]

### Dedicated Hosts
**Best for**: Compliance requirements, server-bound software licenses
- Entire physical server dedicated to you
- Address compliance requirements
- Use existing server-bound software licenses

### Dedicated Instances
**Best for**: Isolation requirements
- No other customers share your hardware
- May share hardware with other instances in same account

### Capacity Reservations
**Best for**: Ensuring capacity availability
- Reserve On-Demand capacity in specific AZ for any duration
- No time commitment required
- Combine with Regional Reserved Instances and Savings Plans for billing discounts
- Charged at On-Demand rate whether you run instances or not

## Networking

### Private vs Public IP (IPv4)

#### Overview
- **IPv4**: Most common format, allows 3.7 billion different addresses
- **IPv6**: Newer format, solves IoT problems
- **Format**: [0-255].[0-255].[0-255].[0-255]

#### Public IP
- Can be identified on the internet (WWW)
- Must be unique across the internet
- Can be geo-located

#### Private IP
- Can be identified on private network only
- Must be unique within the private network
- Different networks can have same private IPs
- Connect to internet using NAT + Internet Gateway

### Elastic IP
- **Problem**: EC2 instances change public IP when stopped/started
- **Solution**: Elastic IP provides fixed public IPv4 address
- Remains yours as long as you don't delete it
- Can attach to only 1 instance at a time
- **Limit**: Maximum 5 Elastic IPs per account

### Placement Groups

Control EC2 instance placement strategy using placement groups:

#### Cluster Placement Group
- **Purpose**: Clusters instances into low-latency group in single AZ
- **Pros**: Great network performance (10 Gbps bandwidth)
- **Cons**: If AZ fails, all instances fail
- **Use cases**: Big data jobs, applications needing extremely low latency and high network throughput

![[Pasted image 20250717094855.png]]

#### Spread Placement Group  
- **Purpose**: Spreads instances across underlying hardware (max 7 instances per group per AZ)
- **Pros**: Can span multiple AZs, reduces risk, instances on different physical hardware
- **Cons**: Limited to 7 instances per AZ per placement group
- **Use cases**: Applications requiring maximum high availability, critical apps needing isolation

#### Partition Placement Group
- **Purpose**: Spreads instances across many partitions within AZ, scales to 100s of instances
- **Pros**: Up to 8 partitions per AZ, up to 100s of EC2 instances, spans multiple AZs, each partition protected from failure
- **Use cases**: HDFS, HBase, Cassandra, Kafka

### Elastic Network Interfaces (ENI)

Logical component in VPC representing a virtual network card:

#### Attributes
- Primary private IPv4, one or more secondary IPv4
- One Elastic IPv4 per private IPv4  
- One public IPv4
- One or more security groups
- A MAC address

#### Characteristics
- Can be created independently
- Attach on the fly to EC2 instances for failovers
- AZ scoped

![[Pasted image 20250717095620.png]]

## Advanced Features

### EC2 Hibernate

#### How it Works
- **RAM state preserved**: RAM is saved to root EBS volume
- **Faster boot**: Instance boot much faster (RAM restored from EBS)
- **Requirements**: Root EBS volume must be encrypted
- **Limitations**: Hibernate no more than 60 days
- **Supported**: Many instance families

#### Use Cases
- Long-running processes
- Saving RAM state
- Services that take time to initialize

## Storage

### EBS (Elastic Block Store)

#### Overview
- Network drive attachable to instances while running
- Allows data persistence even after instance termination  
- Can only be mounted to one instance at a time (except Multi-Attach)
- AZ scoped (cannot attach to instance in different AZ)
- **Free tier**: 30GB SSD

#### EBS Snapshots
- Backup at point in time
- Not necessary to detach volume for snapshot
- Can copy snapshots across AZ or Region

![[Pasted image 20250718221418.png]]

#### Snapshot Features
- **EBS Snapshot Archive**: Move to archive tier, 75% cheaper, 24-72 hrs to restore
- **Recycle Bin**: Setup rules to retain deleted snapshots (1 day to 1 year retention)
- **Fast Snapshot Restore (FSR)**: Force full initialization, no latency on first use (expensive)

![[Pasted image 20250718220700.png]]

#### Volume Deletion Behavior
- **Root EBS volume**: Deleted by default when instance terminates
- **Additional EBS volumes**: Not deleted by default
- **Control**: Can be modified via AWS Console/CLI

### EBS Volume Types

#### SSD-Based Volumes

##### General Purpose SSD (gp2/gp3)
- **Use cases**: System boot volumes, virtual desktops, development environments
- **Size**: 1GB - 16TB
- **Cost**: Cost-effective with low latency

**gp3 (Latest Generation)**:
- Baseline: 3,000 IOPS (max 16,000)
- Throughput: 125 MiB/s (max 1,000 MiB/s)

**gp2 (Previous Generation)**:
- Small volumes can burst IOPS to 3,000
- Size and IOPS linked (3 IOPS per GB)
- Max IOPS: 16,000 at 5,334GB

##### Provisioned IOPS SSD (io1/io2)
- **Use cases**: Critical business applications, databases needing sustained IOPS
- **Multi-Attach**: Can attach same volume to multiple EC2 instances (up to 16)

![[Pasted image 20250718223359.png]]

**io1 (4GB - 16TB)**:
- Max PIOPS: 64,000 (Nitro instances), 32,000 (others)
- Can increase PIOPS independently from storage size

**io2 Block Express (4GB - 64TB)**:
- Sub-millisecond latency
- Max PIOPS: 256,000 (1,000:1 IOPS:GB ratio)
- Supports EBS Multi-attach

#### HDD-Based Volumes

##### Throughput Optimized HDD (st1)
- **Use cases**: Big Data, Data Warehouses, Log Processing
- **Size**: 125GB - 16TB
- **Performance**: Max throughput 500 MiB/s, max IOPS 500
- **Note**: Cannot be boot volume

##### Cold HDD (sc1)  
- **Use cases**: Infrequently accessed data, lowest cost scenarios
- **Size**: 125GB - 16TB
- **Performance**: Max throughput 250 MiB/s, max IOPS 250
- **Note**: Cannot be boot volume

#### Boot Volume Compatibility
Only gp2/gp3 and io1/io2 Block Express can be used as boot volumes.

### EC2 Instance Store

#### Characteristics
- **Performance**: High-performance hardware disk, better I/O than EBS
- **Persistence**: Ephemeral storage (lost if instance stopped)
- **Use cases**: Buffer, cache, scratch data, temporary storage
- **Risk**: Data loss if hardware fails
- **Responsibility**: Backup and replication are your responsibility

### EBS Encryption

#### What Gets Encrypted
- Data at rest inside the volume
- Data in flight between instance and volume  
- All snapshots created from encrypted volume
- All volumes created from encrypted snapshot

#### Technical Details
- Encryption/decryption handled transparently
- Minimal impact on latency
- Uses AWS KMS keys (AES-256)
- Copying unencrypted snapshot allows encryption

#### How to Encrypt Unencrypted EBS Volume
1. Create EBS snapshot of the volume
2. Encrypt the snapshot (copy it with encryption)
3. Create new EBS volume from encrypted snapshot
4. Attach encrypted volume to instance

## AMI (Amazon Machine Image)

### Overview
Customization of EC2 instance including:
- Your own software and configurations
- Operating System
- Monitoring tools
- Faster boot/configuration time (software pre-packaged)

### Characteristics
- Built for specific region (can be copied across regions)
- Can launch from: Public AMI, Your own AMI, AWS Marketplace AMI

### AMI Creation Process
1. Start EC2 instance and customize it
2. Stop the instance (for data integrity)
3. Build AMI (also creates EBS snapshots)
4. Launch instances from the AMI

![[Pasted image 20250718222044.png]]

