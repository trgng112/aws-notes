# EFS - Elastic File System

## Overview
- **Managed NFS**: Network file system that can be mounted on many EC2 instances
- **Multi-AZ Support**: Works with EC2 instances across multiple Availability Zones
- **Scalability**: Highly available, scalable, but expensive (3x cost of gp2)
- **Payment Model**: Pay-per-use with automatic scaling
- **Use Cases**: Content management, web serving, data sharing, WordPress

![[Pasted image 20250718225054.png]]

## Key Features

### Protocol and Compatibility
- **Protocol**: Uses NFSv4.1 protocol
- **OS Compatibility**: Compatible with Linux-based AMI (**Not Windows**)
- **File System**: POSIX file system with standard file API
- **Access Control**: Use security groups to control access to EFS

### Security and Encryption
- **Encryption at Rest**: Uses AWS KMS for encryption
- **Network Security**: Controlled via security groups

### Scalability
- **Automatic Scaling**: File system scales automatically with no capacity planning required
- **Pay-per-Use**: Only pay for storage actually used

## Performance Configuration

### EFS Scale Capabilities
- **Concurrent Clients**: Supports 1000s of concurrent NFS clients
- **Throughput**: Up to 10 GB/s throughput
- **Storage Capacity**: Can grow to Petabyte scale automatically

### Performance Modes
*Note: Set at EFS creation time and cannot be changed*

#### General Purpose (Default)
- **Best for**: Latency-sensitive use cases
- **Use cases**: Web servers, CMS, general applications
- **Characteristics**: Lower latency, moderate throughput

#### Max I/O
- **Best for**: High throughput, highly parallel workloads
- **Use cases**: Big data processing, media processing
- **Trade-off**: Higher latency but greater throughput and parallelism

### Throughput Modes

#### Bursting Mode
- **Base Performance**: 1TB = 50 MiB/s baseline
- **Burst Capability**: Can burst up to 100 MiB/s
- **Best for**: Workloads with predictable patterns

#### Provisioned Mode
- **Control**: Set throughput regardless of storage size
- **Example**: 1 GiB/s throughput for 1TB storage
- **Best for**: Applications requiring consistent high throughput

#### Elastic Mode
- **Auto-scaling**: Automatically scales throughput based on workloads
- **Performance**: Up to 3 GiB/s for reads and 1 GiB/s for writes
- **Best for**: Unpredictable workloads with varying performance needs

## Storage Classes

### Storage Tiers

#### Standard Tier
- **Purpose**: Frequently accessed files
- **Performance**: Highest performance for regular access patterns

#### Infrequent Access (EFS-IA)
- **Cost Model**: Lower storage price but cost to retrieve files
- **Best for**: Files accessed less frequently

#### Archive Tier
- **Purpose**: Rarely accessed data (few times per year)
- **Savings**: 50% cheaper than Standard tier
- **Best for**: Long-term archival and compliance

#### Lifecycle Management
- **Automation**: Implement lifecycle policies to automatically move files between storage tiers
- **Cost Optimization**: Optimize costs based on access patterns

### Availability and Durability Options

#### Standard (Multi-AZ)
- **Durability**: Multi-AZ deployment
- **Best for**: Production workloads requiring high availability
- **Redundancy**: Data replicated across multiple Availability Zones

#### One Zone
- **Deployment**: Single Availability Zone
- **Best for**: Development environments and cost-sensitive workloads
- **Backup**: Backup enabled by default
- **Compatibility**: Works with Infrequent Access (EFS One Zone-IA)
- **Cost Savings**: Over 90% cost savings compared to Standard

## EFS vs EBS Comparison

### EBS (Elastic Block Storage)
- **Attachment**: One instance only (except Multi-Attach on io1/io2)
- **Scope**: Locked to specific Availability Zone
- **Performance Scaling**:
  - **gp2**: I/O increases with disk size
  - **gp3 and io1**: Can increase I/O independently of size

#### EBS Migration Process
- Take a snapshot of the volume
- Restore snapshot to another AZ
- **Important**: EBS backups use I/O and shouldn't run during high traffic periods

![[Pasted image 20250718231210.png]]

#### EBS Instance Termination
- Root EBS volumes are terminated by default when EC2 instance terminates
- This behavior can be disabled if needed

### EFS (Elastic File System)
- **Attachment**: Can mount on 100s of instances across multiple AZs
- **Sharing**: Perfect for sharing website files across instances
- **OS Support**: Linux instances only
- **Cost**: Higher price than EBS
- **Optimization**: Can leverage storage tiers for cost savings

![[Pasted image 20250718231154.png]]

## Key Takeaways

### When to Use EFS
- Need to share files across multiple EC2 instances
- Require cross-AZ file sharing
- Building scalable web applications
- Content management systems
- Development environments requiring shared storage

### When to Use EBS
- Single instance storage needs
- High-performance databases
- Boot volumes
- When Windows compatibility is required

### Remember the Storage Options
- **EFS**: Network file system, multi-instance, Linux-only
- **EBS**: Block storage, single instance (mostly), cross-platform
- **Instance Store**: Temporary, high-performance, ephemeral storage