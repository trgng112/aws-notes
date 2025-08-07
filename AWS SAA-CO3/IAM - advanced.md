[[IAM]]
## AWS Organizations
- Global service
- Allows to manage multiple AWS accs
- main account is the management account
- Other accounts are member accs
- Member accs can only be part of one org
- Consolidated Billing across all accounts - single payment method
- Pricing benefits from aggregated usage (volume discount for EC2,S3)
- **Shared reserved instances and Saving Plan discounts across accounts**
- API is available to automate AWS account  creation
- ![[Pasted image 20250806175007.png]]
- ![[Pasted image 20250806175041.png]]
- Advantages
	- Multi Accs vs One Account Multi VPC
	- Use tagging standards for billing purposes
	- Enable CloudTrail on all accs, send logs to central S3 Account
	- Send CW logs to central logging account
	- Establish Cross Account Roles for admin purposes
- Security: Service Control Policies (SCP)
	- IAM policies applied to OU or Accounts to restrict Users and Roles
	- They do not apply to the management account (full admin power)
	- Must have an explicit allow from the root through each OU in the direct path to the target account (does not allow anything by default - like IAM)
	- ![[Pasted image 20250806175342.png]]

## IAM Conditions
- aws:SourceIP: restrict the client IP **from** which the API calls are being made
- aws:RequestedRegion: restrict the region the api calls are made to
- ![[Pasted image 20250806180007.png]]
- ![[Pasted image 20250806180052.png]]
- ![[Pasted image 20250806180127.png]]
- ![[Pasted image 20250806180204.png]]

## IAM Roles & resource based policies
- Cross account:
	- attaching a resource-based policy to a resource ( s3 bucket policy)
	- OR using a role as a proxy
	- ![[Pasted image 20250806180309.png]]
- When you assume a role (user ,application or service), you give up your original permissions and take the permissions assigned to the role
- When using a resource-based policy, the principal doesn't have to give up his permissions
- Example: User in Account A needs to scan a DynamoDB table in Account A and dump it in a.n S3 bucket in Account 3
- Supported by: S3 buckets, sns topics, sqs queues,...

## Eventbridge - Security
- When a rule runs, it needs permissions on the target
- Resource-based policy: lambda, sns,sqs,s3 buckets, API Gateway
- IAM role: Kinesis stream, EC2 auto scaling, systems Manager, Run Command, ECS tasks
- ![[Pasted image 20250806180552.png]]

## IAM Permission Boundaries
- IAM Permission Boundaries are supported for users and roles (not groups)
- Advanced feature to use a managed policy to set the maximum permissions an IAM entity can get
- ![[Pasted image 20250806180714.png]]
- Can be used in combinations of AWS organizations SCP
- Use case:
	- Delegate responsibilities to non admins within their permission boundaries, for example create new IAM users
	- Allow developers to self-assign policies and manage their own permissions, while making sure they cant escalate their privileges (make themselves admin)
	- Useful to restrict one specific user (instead of a whole account using Org & SCP)
	- ![[Pasted image 20250806181027.png]]
	- ![[Pasted image 20250806181040.png]]
## IAM Identity Center
- one login (single sign on) for all your
	- **AWS accounts in AWS Orgs**
	- Business cloud app (saleforce,box,microsoft 365)
	- SAML2.0 - enabled applications
	- EC2 Windows Instances
- Identity providers
	- Built-in identity store in IAM Identity center
	- 3rd party: AD, oneLogin,Okta
- ![[Pasted image 20250806181402.png]]
- ![[Pasted image 20250806181433.png]]
### Fine-Grained Permissions and Assignments
- Multi- Account Permissions
	- Manage access across AWS accs in your AWS org
	- Permission sets- a collection of one or more IAM policies assigned to users and groups to define AWS access
- Application Assignments
	- SSO access to many SAML 2.0 business apps (salesforce, box, microsoft 365)
	- Provide required URLs, certs, metadata
- Attribute-Based Access Control (ABAC)
	- Fine-grained permissions based on users attributes stored in IAM identity center identity store
	- Example: cost center, title , locale,...
	- Use case: define permissions once, then modify AWS access by changing the attributes
- ![[Pasted image 20250806181807.png]]

## AWS Directory Services
- ![[Pasted image 20250807115613.png]]
- **AWS Managed Microsoft AD**
	- Create your own AD in AWS, manage users locally, supports MFA
	- Establish 'trust' connections with your on-premise AD
- **AD Connector**
	- Directory Gateway (proxy) to redirect to on-premise AD, supports MFA
	- Users are manged on the on-premise AD
- **Simple AD**
	- AD-compatible managed directory on AWS
	- Cannot be joined with on-premise AD
- ![[Pasted image 20250807115836.png]]

### Identity Center - Active Directory Setup
- Connect to an AWS Managed Microsoft AD (Directory Service)
	- Integration is out of the box
	- ![[Pasted image 20250807115940.png]]
- **Connect to a Self-Managed Directory**
	- Create Two-way Trust Relationship using AWS Managed Microsoft AD
	- Create an AD Connector
	- ![[Pasted image 20250807120024.png]]

## AWS Control Tower
- Easy way to **set up and govern a secure and compliant multi account AWS environment based on best practices
- AWS control tower uses AWS Organizations to create acconuts
- Benefits:
	- Automate the setup of your env in a few clicks
	- Automate ongoing policy management using guardrails
	- Detect policy violations and remediate them
	- Monitor compliance through an interactive dashboard
### Guardrails
- Provides ongoing governance for your Control Tower Env (AWS accs)
- **Preventive Guardrail - using SCPs** (Restrict Regions across all your accounts)
- **Detective Guardrail - Using AWS Config**  (identify untagged resources)
- ![[Pasted image 20250807120418.png]]
- 