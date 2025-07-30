
## ECS
### EC2 Launch Type
- ECS = Elastic Container SErvice
- Launch Docker containers on AWS = launch **ECS Tasks** on ECS Clusters
- **EC2 Launch Type: you must provision & maintain the infrastructure (the EC2 instances)**
- Each EC2 instance must run the ECS Agent to register in the ECS cluster
- AWS takes care of starting/stopping containers
- ![[Pasted image 20250730194542.png]]

### Fargate Launch Type 
- Launch Docker containers on AWS
- **You do not provision the infrastructure (no EC2 instances to manage)**
- **It's all Serverless**
- You just create task definition
- AWS just run ECS Tasks for you based on the CPU/RAM you need
- To scale, just increase the number of tasks. Simple - no more EC2 instances
- ![[Pasted image 20250730194721.png]]

### IAM Roles for ECS
- EC2 Instance Profile (EC2 Launch Type only)
	- Used by the ECS agent
	- Makes API calls to ECS service
	- Send container logs to CloudWatch logs
	- Pull docker image from ECR
	- Reference sensitive data in Secrets Manager or SSM Parameter Store
- ECS Task Role
	- Allows each tasks to have a specific role
	- Use different roles for the different ECS Services you run
	- Task Role is defined in the task definition
	- ![[Pasted image 20250730194903.png]]

### Load Balancer Integrations
- ALB supported and works for most use cases
- NLB recommended only for high throughput / high performance use cases, or to pair it with AWS Private Link
- CLB supported but not recommended (no advanced features - no Fargate)
- ![[Pasted image 20250730195140.png]]

### Data Volumes EFS
- Mount EFS file systems onto ECS tasks
- Works for both **EC2** and **Fargate** launch types
- Task running in any AZ will share the same data in the EFS file system
- **Fargate + EFS = Serverless**
- Use cases: persistent multi-AZ shared storage for containers
- Notes:
	-  S3 cannot be mounted as a file system
- ![[Pasted image 20250730195305.png]]

### ECS Service Auto Scaling
- Automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses **AWS Application Auto Scaling**
	- ECS Service Average CPU utilization
	- ECS Service Average Memory Utilization - Scale on RAM
	- ALB Request Count Per Target - metric coming from the ALB
- **Target tracking** - scaled based on target value for a specific CloudWatch metric
- **Step Scaling** - scale based on a specified Cloudwatch Alarm
- **Scheduled Scaling** scale based on a specified date/time (predictable changes)
- ECS Service Auto Scaling (task level) # EC2 Auto Scaling (EC2 instance level)
- Fargate Auto Scaling is much easier to setup (because **Serverless**)

#### EC2 Launch Type - Auto Scaling EC2 instances
- Accommodate ECS Service Scaling by adding underlying EC2 Instances
- **Auto Scaling Group Scaling**
	- Scale your ASG based on CPU utilization
	- Add EC2 instances over tiem
- ECS Cluster Capacity Provider
	- Used to automatically provision and scale the infrastructure for your ECS Tasks
	- Capacity Provider paired with an Auto Scaling Group
	- Add EC2 Instances when you're missing capacity (CPU,RAM)
- Example:
	- ![[Pasted image 20250730200625.png]]

### ECS tasks invoked by Event Bridge
- ![[Pasted image 20250730200719.png]]

### ECS tasks invoked by Event Bridge Scheduled
![[Pasted image 20250730200751.png]]
### ECS SQS Queue example
![[Pasted image 20250730200816.png]]
### Intercept Stopped Tasks using EventBridge
![[Pasted image 20250730200843.png]]


## ECR
- Elastic Container Registry
- Store and Manage Docker images on AWS
- **Private and Public** repos (**Amazon ECR Public Gallery**)
- Fully integrated with ECS, backed by S3
- Access is controlled through IAM (permission errors => policy)
- Supports image vulnerability scanning, versioning, image tags, image lifecycle,...
- ![[Pasted image 20250730201126.png]]

## EKS
- Elastic Kubernetes Service
- It is a way to launch managed Kubernetes clusters on AWS
- Kubernetes is an open source system for automatic deployment, scaling and management of containerized (usually Docker) application
- It's an alternative to ECS, similar goal but different API
- Supports **EC2** for deploy worker nodes or **Fargate** to deploy serverless containers
- Use case: if your company is already using Kubernetes on premises or in another cloud, and wants to migrates to AWS using Kubernetes
- **Kubernetes is cloud-agnostic** (can be used in any cloud - Azure GCP)
- ![[Pasted image 20250730202612.png]]

### Node Types
- Managed Node Groups
	- Creates and manages Nodes (EC2 instances) for you
	- Nodes are part of an ASG managed by EKS
	- Supports On-Demand or Spot Instances
- Self Managed Nodes
	- Nodes created by you and registered to the EKS cluster and managed by an ASG
	- You can use prebuilt AMI - Amazon EKS Optimized AMI
	- Supports On Demand or Spot Instances
- AWS Fargate
	- No maintenance required, no nodes managed

### Data Volumes
- Need to specify StorageClass manifest on your EKS cluster
- Leverages a **Container Storage Interface** (CSI) compliant driver
- Support for: EBS EFS FSx Lustre, FSX NetApp ONTAP

## App Runner
- Fully managed service that makes it easy to deploy web apps and APIs at scale
- No infra experience required
- Start with your source code or container image
- Automatically builds and deploy the webapp
- Automatic scaling, highly available, load balancer, encryption
- VPC access support
- Connect to database, cache, and message queue services
- Use case: web apps, APIs, microservices, rapid production deployments
- ![[Pasted image 20250730203250.png]]

## App2Container
- CLI tool for migrating and modernizing Java and .NET web apps into docker containers
- **Lift and shift** your apps running in on-premises bare metal, virtual machines, or in any cloud to AWS
- Accelerate modernization, no code changes, migrate legacy apps
- Generate CloudFormation templates (computes,network)
- Register generated Docker containers to ECR
- Deploy to ECS EKS or App Runner
- Support pre-built CICD pipelines
- ![[Pasted image 20250730203731.png]]
- 