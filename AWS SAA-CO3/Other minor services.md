# Cloud Formation
- Is a declarative way of outlining your AWS Infrastructure, for any resources (Most of them are supported).
- For example, within a CloudFormation template:
	- SG
	- 2 EC2 instances using this SG
	- S3 Bucket
	- Load Balancer (ELB) in front of these machines
- Then CloudFormation creates those for you, in the **right order** with the **exact configs** that you specify

### Benefits
- Infrastructure at code
	- No resources are manually created, which is excellent for control
	- Changes to the infrastructure are reviewed through code
- Cost
	- Each resources within the stack is tagged with an ID so you can easily see how much a stack costs you
	- You can estimate the costs of your resources using CloudFormation template
	- Saving strat: in Dev, you could automation deletion of templates at 5PM and recreated at 8 AM safely
- Productivity:
	- Ability to destroy and re-create an infra on the cloud on the fly
	- Automated generation of Diagram for you templates
	- Declarative programming (no need to figure out ordering and orchestration)
- Don't reinvent the wheel
	- Leverage existing templates on the web
	- Leverage the documentation
- Support (almost) all AWS resources:
	- Everything in this course is supported
	- You "custom resources" for resources that are not supported

### CloudFormation + Infrastructure Composer
- We can see all the resources
- WE can see the relations between the components
- ![[Pasted image 20250812161105.png]]

### Service Role
- IAM Role that allows CloudFormation to create/update/delete stack resources on your behalf
- Give ability to users to create/update/delete the stack resources event if they don't have permissions to work with the resources in the stack
- Use case:
	- You want to achieve the least privilege principle
	- but you don't want to give the user all the required permissions to create the stack resources
- **User must have iam:PassRole** permissions
- ![[Pasted image 20250812161636.png]]

## Amazon Simple Email Service SES
- Fully managed service to send emails securely,globally and at scale
- Allow inbound/outbound emails
- Reputation dashboard, performance insights, anti-spam feedback
- Provides statistics such as email deliveries, bouncevs, feedback loop results, email open
- Supports DomainKeys Identified Mail (DKIM) and Sender Policy Framework (SPF)
- Flexible IP deployment: shared,dedicated, and customer-owned IPs
- Send emails using your application using AWS Console, APIs, or SMTP
- Use cases: transactional, marketing, bulk email communications

![[Pasted image 20250812161851.png]]
## Amazon Pinpoint
- Scalable **2-way (outbound/inbound)** marketing communication service
- Supports email,SMS,push,voice,in app messaging
- Ability to segment and personalize messages with the right content to customers
- Possibility to receive replies
- Scales to billions of messages per day
- Use case: run campaigns by sending marketing, bulk, transactional SMS messages
- **Versus SNS and SES**
	- In SNS and SES, you  managed each message's audience, content and delivery schedule
	- In Pinpoint, you create message templates, delivery schedules, highly targeted segments, and full campaigns
- ![[Pasted image 20250812162058.png]]

## Systems Manager 

###  SSM Session Manager
- Allows you to start a secure shell on your EC2 and on premises servers
- **No SSH access, bastion hosts, or SSH keys needed**
- No port 22 needed (better security)
- Supports Linux, macOS and Windows
- Send session log data to S3 or CW logs
- ![[Pasted image 20250812162203.png]]

### Run Command
- Execute a document (=script) or just run a command
- Run command across multiple instances (using resource groups)
- No need for SSH
- Command Output can be shown in the AWS Console, sent to S3 bucket or CW Logs
- Send notifications to SNS about command status (in progress, success, failed)
- Integrated with IAM and CloudTrail
- Can be invoked with EventBridge
- ![[Pasted image 20250812162518.png]]

### Patch Manager
- Automates the process of patching managed instances
- OS updates, application updates, security updates,
- supports ec2 instances and on premises servers
- supports linux,macos, windows
- patch on demand or on a schedule using **Maintenance Windows**
- Scan instances and generate patch compliance report (missing patches)
- ![[Pasted image 20250812162620.png]]

### Maintenance Windows
- Defines a schedule for when to perform actions on your instances
- Example: OS patching, updating drivers, installing software,...
- Maintenance Window contains
	- Schedule
	- Duration
	- Set of registered instances
	- Set of registered tasks
- ![[Pasted image 20250812162708.png]]
### Automation
- Simplified common maintenance and deployment tasks of EC2 instances and other AWS resources
- Examples: restart instances, create an AMI, EBS snapshot
- **Automation Runbook** - SSM Documents to define actions performed on your EC2 instances or AWS resources (pre-defined or custom)
- Can be triggered using:
	- Manually using AWS Console, AWS CLI or SDK
	- Event Bridge
	- On a schedule using Maintenance Windows
	- AWS Config for rules remediations
- ![[Pasted image 20250812162842.png]]
## AWS Cost Explorer
- Visualize, understand and manage aws costs and usage over time
- create custom reports that analyze cost and usage data
- Analyze your data at a high level: total costs and usage across all accounts
- Monthly,hourly, resource level granularity
- Choose an optimal **Saving Plan** (to lower prices on your bill)
- **Forecast usage up to 12 months based on previous usage**

![[Pasted image 20250812163006.png]]
![[Pasted image 20250812163014.png]]
![[Pasted image 20250812163029.png]]

![[Pasted image 20250812163052.png]]

## AWS Cost Anomaly Detection
- Continuously monitor your cost and usage using ML to detect unusual spends
- It learns your unique, historic spend patterns to detect one time cost spike and / or continuous cost increases (you don't need to define thresholds)
- Monitor AWS services, member accounts, cost allocation tags, or cost categories
- Sends you the anomaly detection report with root-cause analysis
- Get notified with individual alerts or daily/weekly summary (using SNS)
- ![[Pasted image 20250812163213.png]]
## AWS Outposts
- **Hybrid Cloud** : business that keep an on-premises infra alongside a cloud infra
- Therefore , two ways of dealing with IT systems:
	- One for the AWS cloud (AWS Console, CLI, AWS APIs)
	- One for their on-premise infra
- **AWS Outposts are "server racks"** that offers the same AWS infra, services, APIs and tools to build your own apps on premises just as in the cloud
- **AWS will setup and manage "Outpost Racks"** within your on premises infra and you can start leveraging AWS services on premises
- **You are responsible for the Outpost Racks physical security **
![[Pasted image 20250812163403.png]]

### Benefits:
 - Low latency access to on premises systems
 - Local data processing
 - Data residency
 - Easier migration from on premises to the cloud
 - Fully managed service
 - Some service that works on outpost
 - ![[Pasted image 20250812163508.png]]
 
## AWS Batch
- **Fully managed** batch processing at **any scale**
- Efficiently run 100,000s of computing batch jobs on AWS
- a 'batch' job is a job with a start and an end (opposed to continuous)
- Batch will dynamically launch  **EC2 instances and Spot Instances**
- AWS Batch provisions the right amount of compute / memory
- You submit or schedule batch jobs and AWS Batch does the rest
- Batch jobs are defined as Docker images and run on ECS
- Helpful for cost optimizations and focusing less on the infra
![[Pasted image 20250812163709.png]]
### Batch vs Lambda
 - Lambda:
	 - Time limit
	 - Limited runtimes
	 - limited temp disk space
	 - Serverless
 - Batch:
	 - No time limit
	 - Any runtime as long as it packaged as a Docker image
	 - Rely on EBS / instance store for disk space
	 - Relies on EC2 (can be managed by AWS)

## AWS AppFlow
- Fully managed integration service that enables you to securely transfer data between **SaaS** apps and AWS
- Sources: SalesForce, SAP, Zendesk, Slack, ServiceNow
- Destinations: **AWS services like S3, Redshift** or non AWS such as SnowFlake and Salesforce
- Frequency: on a schedule, response to events or on demand
- Data transformation capabilities like filtering and validation
- Encrypted over the public internet or privately over AWS PrivateLink
- Don't spend time writing the integrations and leverages APIs immediately
![[Pasted image 20250812163932.png]]
## AWS Amplify
- A set of tools and services that helps you develop and deploy scalable full stack web and mobile apps
- Auth,storage,api,Ci/CD,...
- Connect your source code from github, AWS code commit, bitbucket,gitlab, or upload directly
- ![[Pasted image 20250812164029.png]]
## AWS Instance Scheduler
- AWS solution deployed through CloudFormation (not a service)
- **Automatically start/stop your AWS services to reduce costs (up to 70%)**
- Example: stop company's EC2 instances outside business hours
- **Supports EC2 instances , EC2 Auto Scaling Groups, RDS instances**
- Schedules are managed in a DynamoDB table
- Uses resource's tags and Lambda to stop/start instances
- Supports cross-account and cross-region resources
- 