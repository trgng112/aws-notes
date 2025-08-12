
# Lambda SNS & SQS


![[Pasted image 20250812112343.png]]
![[Pasted image 20250812112348.png]]
![[Pasted image 20250812112358.png]]
![[Pasted image 20250812112410.png]]

![[Pasted image 20250812112418.png]]
![[Pasted image 20250812112425.png]]![[Pasted image 20250812112436.png]]
![[Pasted image 20250812112446.png]]

![[Pasted image 20250812112459.png]]
![[Pasted image 20250812112517.png]]
![[Pasted image 20250812112524.png]]
# High Performance Computing (HPC) on AWS
- Perform genomics, computational chemistry, financial risk modeling,
weather prediction, machine learning, deep learning, autonomous driving

## Data Management & Transfer
- Direct Connect:
	- Move GB/s of data to the cloud over a private secure network
- Snowball & snowmobile:
	- Move PB of data to the cloud
- AWS Datasync
	- Move large amount of data between on premises and s3, efs, fsx for windows

## Compute and Networking
- EC2 instances
	- CPU optimized, GPU optimized
	- Spot instances / spot fleets for cost saving + auto scaling
- Placement groups:
	- **Cluster** for good network performance
	- ![[Pasted image 20250812111915.png]]
- EC2 Enhanced Networking (SR-IOV)
	- Higher bandwidth, higher PPS (packet per second), lower latency
	- Option 1: **Elastic Network Adapter (ENA)** up to 100Gbps
	- Option 2: Intel 82599 VF up to 10Gbps - LEGACY
- **Elastic Fabric Adapter (EFA)**
	- Improved ENA for HPC, only works for Linux
	- Great for inter-note communications, tightly coupled workloads
	- Leverages Message passing Interface (MPI) standard
	- **Bypasses the underlying Linux OS to provide low-latency,reliable transport**
## Storage
- Instance - attached storage:
	- **EBS:** scale up to 256,000 IOPS with io2 Block Express
	- **Instance Store** scale to millions of IOPS, linked to EC2 instance, low latency
- Network storage:
	- S3: Large blob, not a file system
	- EFS: scale IOPS based on total size, or use provisioned IOPS
	- FSx for Lustre:
		- HPC  optimized distributed file system, millions of IOPS
		- Backed by S3

## Automation and Orchestration
- AWS Batch
	- AWS Batch supports multi-node parallel jobs, which enables you to run singlejobs that span multiple EC2 instances.
	- Easily schedule jobs and launch EC2 instances accordingly
- AWS ParallelCluster
	-  Open-source cluster management tool to deploy HPC on AWS
	-  Configure with text files
	- Automate creation of VPC, Subnet, cluster type and instance types
	- Ability to enable EFA on the cluster (improves network performance)



# Highly Available EC2 instance
- ![[Pasted image 20250812112559.png]]
- ![[Pasted image 20250812112604.png]]
- ![[Pasted image 20250812112608.png]]
- 