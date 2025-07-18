### EFS - Elastic File System
- Managed NFS - network file system can be mounted on many EC2
- works with ec2 instances in multiple AZ
- highly available,scalable,expensive (3 x gp2), pay per use
- content management, web serving, data sharing, wordpress
- use NTFSV4.1 protocol
- use security group to controll access to EFS
- compatible with Linux based AMI (**Not Windows**)
- Encryption at rest usign KMS
- POSIX file system that has standard file API
- File system scales automatically, pay-per-use,no capacity planning
- ![[Pasted image 20250718225054.png]]

#### Performance and Storage classes
- EFS Scale
	- 1000s of concurrent NFS clients, 10GB /s throughput
	- Grow to Petabyte scale network file system, automatically
- Performance mode (set at EFS creation time)
	- General purpose (default): latency-sensitive use case (web server, CMS,etc...)
	- Max I/O - higher latency, throughput, highly parallel (big data, media processing)
- Throughput Mode
	- Bursting - 1TB = 50MiB/s + burst up to 100 mib/s (not important)
	- Provisioned - set your throughput regardless of storage size , ex: 1GiB/s for 1 TB storage
	- Elastic: automatically scales throughput up or down based on your workloads
		- Up to 3GiB/s for reads and 1GiB/s for writes
		- for unpredictable workloads

- Storage classes
	- Storage Tiers
		- Standard: frequently accessed files
		- Infrequent Access (EFS-IA) cost to retrieve files, lower price to stgore.
		- Archive: rarely accessed data (few times each year), 50% cheaper
		- Implement lifecycle policies to move files between storage tiers
	- Availability and durability
		- Standard: multi az, great for prod
		- One zone: one AZ, great for dev, backup enabled by default, compatible with IA (EFS One Zone - IA)
	- Over 90% in cost savings

#### EFS vs EBS - Elastic Block Storage

- EBS
	- one instance (except for multi attach io1/ io2)
	- locked by AZ
	- gp2: IO increase if the disk size increases
	- gp3 and io1: can increase IO independently
	- To migrate EBS volume across AZ
		- take a snapshot
		- restore snapshot to another AZ
		- EBS backup use IO and shouldn't run them while your application is handling a lot of traffic
		- ![[Pasted image 20250718231210.png]]
	- Root EBS volumes of instances get terminated by default if the EC2 instance is terminated ( can be disabled)
- EFS
	- Mounting 100s instances across AZ
	- EFS share website files
	- only for linux instances
	- has higher price than EBS
	- can leverage storage tiers for cost saving
	- remember EFS vs EBS vs Instance store![[Pasted image 20250718231154.png]]