[[S3]] links to this
### AWS Snowball
- Highly-secure, portable devices to collect and process data at the edge, and migrate data into and out of AWS
- Helps migrate up to Petabytes of data
- ![[Pasted image 20250729141206.png]]
- Data migrations:
	- Challenges:
		- Limited connectivity
		- Limited Bandwidth
		- High network cost
		- Shared bandwidth (can't maximize the line)
		- Connection stability
- AWS Snowball: offline devices to perform data migrations, if it takes more than a week to transfer over the network, use snowball devices
- ![[Pasted image 20250729141334.png]]

#### Edge Computing
- Process data while it's being created on an edge location
- These locations may have limited internet and no access to computing power
- WE setup a Snowball Edge device to do edge computing
	- Snowball Edge Compute Optimized (dedicated for that use case) and storage optimized
	- Run EC2 Instances or Lambda functions at the edge
- Use cases: preprocess data, machine learning, transcoding media
#### Solution Architecture: snowball into Glacier
- Snowball cannot import to Glacier directly
- use S3 first, in combination with an s3 lifecycle policy
- ![[Pasted image 20250729141715.png]]


### Amazon FSx 
- Launch 3rd party high-performance file system on AWS
- fully managed service
- ![[Pasted image 20250729141745.png]]

##### Amazon FSx for Windows (file server)
- FSx for Windows is a fully managed Windows file system share drive
- Supports SMG protolcol & Windows NTFS
- Microsoft ACtive Directory integration,ACLs, user quotas,
- Can be mounted on Linux EC2 instances
- Supports Microsoft's Distributed File System (DFS) Namespaces (group file across multiple FS)
- Scale up to 10s of GB/s, millions of IOPS, 100s PB of data
- Storage options:
	- SSD: latency sensitive workloads (db,media processing)
	- HDD: broad spectrum of workloads (home directory, cms)
- Can be accessed from your on-premises infrastructure (VPN or Direct Connect)
- Can be configured to be multi az
- data is backed up daily to S3

##### Amazon FSx for Lustre
- Lustre is a type of parallel distributed file system, for large scale computing
- the name Lustre is derived from "Linux" and "cluster"
- Machine Learning, High Performance Computing (HPC)
- Video Processing, Financial Modeling, Electronic design automation
- Scales up to 100s GB/s, millions of IOPS, sub-ms latencies
- Storage Options:
	- SSD: low-latency, IOPS intensive workloads, small and random file operations
	- HDD- throughput intensive workloads, large & sequential file operations
- Seamless integration with S3
	- Can read s3 as a file system (through FSx)
	- Can write the output of the computations back to S3
- can be used from on-premises servers (vpn or direct connect)


#### FSx FIle system deployment Options
- Scratch File System
	- Temp storage
	- Data is not replicated (doesnt persist if file server fails)
	- High burst (6x faster, 200mbps per tib)
	- usage: short-term processing , optimize cost
	- ![[Pasted image 20250729142304.png]]
- Persistent File System
	- Long term storage
	- data is replicated within same AZ
	- REplaced failed files within minutes
	- usage: long term processing, sensitive data
	- ![[Pasted image 20250729142354.png]]

#### FSx for NetApp Ontap
- Manged NetApp ONTAP on AWS
- File System compatible with NFS,SMB,iSCSI protocol
- Move workloads running on ONTAP or NAS to AWS
- Works with:
	- Linux,
	- windows,
	- mac
	- VMwarecloud on AWS
	- Amazon workspaces and appstream 2.0
	- EC2,ECS,EKS
- Storage shrinks or grows automatically
- Snapshots, replication, low-cost, compression and data deduplication
- **Point-in-time instantaneous cloning (helpful for testing new workloads)**
- ![[Pasted image 20250729142544.png]]

#### FSx for OpenZFS
- Managed OpenZFS file system on AWS
- File system compatible NFS (v3 v4 v4.1 v4.2)
- Move workloads running on ZFS to AWS
- works with
	- - Linux,
	- windows,
	- mac
	- VMwarecloud on AWS
	- Amazon workspaces and appstream 2.0
	- EC2,ECS,EKS
- Up to 1 mil IOPS with <0.5ms latency
- snapshots , compression and low cost
- **Point-in-time instantaneous cloning (helpful for testing new workloads)**
- ![[Pasted image 20250729142722.png]]





### Storage Gateway
#### Hybrid Cloud for storage
- AWS is pushing for hybrid cloud
	- part of your infra on the cloud
	- other in on-remises
- This can be due to
	- Long cloud migrations
	- Security requirements
	- compliance requirements
	- it strat
- S3 is a proprietary storage technology (unlike EFS / NFS), so how do you expose the S3 data on on-premises? => AWS storage gateway

#### AWS Storage Cloud Native Options 
![[Pasted image 20250729143154.png]]
#### AWS Storage Gateway
- Bridge between on-premises data and cloud data
- Use cases:
	- Disaster recovery
	- backup restore
	- tiered storage
	- on premises cache & low latency files access
	- ![[Pasted image 20250729143254.png]]
- Types of storage gateway:
	- S3 file GW
	- FSx file GW
	- Volume GW
	- Tape GW
##### S3 File Gateway
- Configured S3 buckets are accessible using NFS and SMB protocol
- **most recently used data is cashed in the file gateway**
- Supports s3 standard, s3 Standard IA, S3 One Zone A, S3 intelligent tiering
- **transition to S3 Glacier using a Lifecycle policy**
- Bucket access using IAM roles for each File GW
- SMB Protocol has integration with Active Directory for user authentication 
- ![[Pasted image 20250729143459.png]]

##### FSX File Gateway - deprecated
![[Pasted image 20250729143553.png]]


##### Volume GW
- Block storage using iSCSI protocol backed by S3
- Backed by EBS snapshots which can help restore on premises volumes
- Cache volumes: low latency access to most recent data
- Stored volumes: entire dataset is on premise, scheduled backups to S3
- ![[Pasted image 20250729143652.png]]

##### Tape GW
- Some companies have backup processes using physical tapes
- With tape GW, companies use the same processes but in the cloud
- Virtual Tape Library (VTL) backed by Amazon S3 and Glacier
- Backup data using existing tape-based processes (iSCSI interface)
- Works with leading backup software vendors
- ![[Pasted image 20250729143757.png]]

##### Storage GW - hardware appliance
- Using Storage Gateway means you need on-premises virtualization
- Otherwise you can use a **Storage Gateway Hardware Appliance**
- You can buy it on amazon
- Works with File Gateway, Volume Gateway, Tape GW,
- Has the required CPU,memory,network,SSD cache resources
- Helpful for daily NFS backups in small data centers
- ![[Pasted image 20250729143858.png]]

![[Pasted image 20250729144017.png]]




### AWS Transfer Family
- A fully managed service for files transfer into and out of Amazon S3 or EFS using FTP protocol
- Supported protocols
	- AWS transfer for FTP
	- AWS transfer for FTPS
	- AWS transfer for SFTP
- Managed infrastructure, scalable, reliable, highly available (multi AZ)
- Pay per provisioned endpoint per hour + data transfer in GB
- Integrate with existing authentication system (microsoft active directory, LDAP,OKta, amazon cognito,custom))
- ![[Pasted image 20250729144658.png]]
### AWS Datasync
- Move large amout of data to and from
	- On premises/ other cloud to AWS - **needs agent**
	- AWS to AWS (different storage services) no agent needed
- Can sync to:
	- Amazon S3 (any storage classes - including Glacier)
	- EFS
	- FSX (windows,Lustre,NetApp,OpenZFS)
- Replication tasks can be scheduled hourly, daily, weekly
- File permissions and metadata are preserved (NFS POSIX SMB)
- One agent task  can use 10 GBPS, can setup a bandwidth limit
- ![[Pasted image 20250729144939.png]]
- ![[Pasted image 20250729145130.png]]


### Storage Comparison:
- S3 : Object Storage
- S3 Glacier; Object ARchival
- EBS volumes: network storage for one EC2 instance at a time
- Instance storage: physical storage for your ec2 instance (high IOPS)
- EFS Network FIle system for linux instances, POSIX sistem
- FSX for windows: network file system for windows server
- FSXx for Lustre: High performance computing linux file system
- FSX for netapp ontap: high os compatibility
- FSx for OpenZFS: Managed ZFS file system
- Storage Gateway: S3 and FSx File GW, Volume GW (cached and stored), tape GW
- Transfer Family: FTP FTPS SFTP interface on top of S3 or EFS
- DataSync: schedule data sync from on premises to AWS or AWS to AWS
- Snowcone/snowball/snowmobile: to move large amount of data to the cloud physically
- database: for specific workloads, usually with indexing and querying