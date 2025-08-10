- Encryption in Flight (TLS / SSL)
	- Data is encrypted before sending and decrypted after receiving
	- TLS certificates help with encryption (HTTPS)
	- Encryption in flight ensures no MITM (man in the middle attack) can happen
	- ![[Pasted image 20250807120918.png]]
- Server side Encryption at rest
	- Data is encrypted after being received by the server
	- data is decrypted before being sent
	- It is stored in an encrypted form thanks to a key (usually a data key)
	- The encryption / decryption keys must be managed somewhere and the server must have access to it
	- ![[Pasted image 20250807121047.png]]
- Client side encryption
	- Data is encrypted by the client and never decrypted by the server
	- data will be decrypted by a receiving client
	- the server should not be able to decrypt the data
	- could leverage envelope encryption
	- ![[Pasted image 20250807121137.png]]
## AWS KMS - Key Management Service
- Anytime you hear "encryption" for an AWS services, its most likely KMS
- AWS manages encryption keys for us
- Fully integrated with IAM for authorization
- Easy way to control access to your data
- Able to audit KMS key usage using cloudtrail
- Seamlessly integrated into most AWS services (EBS,S3,RDS,SSM,...)
- **Never ever store your secrets in plaintext, especially your code**
	- KMS Key Encryption also available through API calls (SDK,CLI)
	- Encrypted secrets can be stored in the code / env vars

### KMS Key Types
- **KMS Keys is the new name of KMS Customer Master Key**
- **Symmetric (AES-256 keys)**
	- Single encryption that is used to Encrypt and Decrypt
	- AWS services that are integrated with KMS use symmetric CMKs
	- You never get access to the KMS Key unencrypted (must call KMS API to use)
- **Asymmetric (RSA & ECC key pairs)**
	- Public (encrypt) and Private Key (decrypt) pair
	- Used for Encrypt/Decrypt , or SIgn/verify operations
	- The public key is downloadable, but you cant access the Private Key unencrypted
	- Use case: encryption outside of AWS by users who cant call the KMS API
- Types of KMS Keys:
	- AWS Owned Keys (free): SSE-S3, SSE-SQS, SSE-DDB (default key)
	- AWS Managed Key:free (aws/service-name: aws/rds , aws/ebs)
	- Customer managed keys created in KMS : $1/month
	- Customer managed keys imported $1/month
	- + pay for API call to KMS ($0.03/10000 calls)
- Automatic Key rotation:
	- AWS managed KMS key: automatic every 1 year
	- Customer managed KMS Key (must be enabled) automatic & on demand
	- Imported KMS key: only manual rotation possible using alias

### Copying Snapshots across regions
- ![[Pasted image 20250807121813.png]]
### KMS Key Policies
- Control access to KMS keys, 'similar' to S3 bucket policies
- Difference: you cannot control access without them
- **Default KMS Key Policy**
	- Created if you dont provide a specific KMS Key Policy
	- Complete access to the key to the root user = entire AWS account
- **Custom KMS Key Policy**
	- Define users, roles that can access the KMS key
	- Define who can administer the key
	- Useful for cross-account access of your KMS key

### Copying Snapshots across accounts
- Create a snapshot, encrypted with your own KMS Key (customer managed key)
- **Attach a KMS key policy to authorize cross account access**
- Share the encrypted snapshot
- (in target) Create a copy of the Snapshot, encrypt it with a CMK in your account
- ![[Pasted image 20250807122036.png]]


### KMS Multi-Region Keys
- Identical KMS keys in different Regions that can be used interchangeably
- Multi-region keys have the same key ID, key material, automatic rotation
- Encrypt in one Region and decrypt in other Regions
- No need to re-encrypt or making cross-region API calls
- KMS multi-region are not global (primary + replicas)
- Each multi-region key is managed **independently**
- Use cases: global client-side encryption, encryption on Global DynamoDB, Global Aurora

#### DynamoDB Global Tables and KMS Multi Region keys client side encryption
- We can encrypt specific attributes client side in our dynamoDB  table using **Amazon DynamoDB Encryption Client**
- Combined with Global Tables, the client side encrypted data is replicated to other regions
- If we use a multi region key, replicated in the same region as the DynamoDB Global Table, then clients in these regions can use low-latency API calls to KMS in their region to decrypt the data client-side
- Using client-side encryption we can protect specific fields and guarantee only decryption if the client has access to an API key
- ![[Pasted image 20250807181107.png]]

#### Global Aurora and KMS multi region keys client side encryption
- Use **AWS Encryption SDK**
- Combined with Aurora Global Tables, the client side encrypted data is replicated to other regions
- If we use a multi region key, replicated in the same region as the DynamoDB Global Table, then clients in these regions can use low-latency API calls to KMS in their region to decrypt the data client-side
- ![[Pasted image 20250807181155.png]]
- Using client-side encryption we can protect specific fields and guarantee only decryption if the client has access to an API key, **we can protect specific fields event from db admins**


## S3 Replication with Encryption
- Unencrypted objects and objects encrypted with SSE S3 are replicated by default
- Objects encrypted with SSE-C (customer provided key) can be replicated
- **For objects encrypted with SSE-KMS** you need to enable the option
	- Specify which KMS Key to encrypt the objects within the target bucket
	- Adapt the KMS Key Policy for the target key
	- An IAM Role with kms:Decrypt for the source KMS Key and kms:Encrypt for the target KMS Key
	- You might get KMS throttling errors, in which case you can ask for a Service Quotas increase
- **You can use multi-region AWS KMS Keys, but they are currently treated as independent keys by S3 (the object will still be decrypted and then encrypted**

## AMI Sharing Process Encrypted via KMS
- AMI in Source Account is encrypted with KMS Key from Source Account
- Must modify the image attribute to add a **Launch Permission** which corresponds to the specified target AWS acc
- Must share the KMS Keys used to encrypted the snapshot the AMI references with the target account / IAM role
- The IAM Role / User in the target account must have the permissions to DescribeKey, ReEncrypt*,CreateGrant,Decrypt
- When launching an EC2 Instance from the AMI, optionally the target account can specify a new KMS key in its own account to re-encrypt the volumes
- ![[Pasted image 20250807181708.png]]


## SSM Parameter store
- Secure storage for configs and secrets
- Optional seamless encryption using KMS
- serverless, scalable,durable,easy SDK
- Version tracking of configs/ secrets
- Security through IAM
- Notifications with Amazon EventBridge
- Integration with CloudFormation
- ![[Pasted image 20250807181824.png]]

### Store Hierarchy
- ![[Pasted image 20250807181912.png]]
- ![[Pasted image 20250807181923.png]]
### Parameters Policies (for advanced params)
- Allow to assign a TTL to a param (expiration date) to force updating or deleting sensitive data such as passwords
- Can assign multiple policies at a time
- ![[Pasted image 20250807182018.png]]



## AWS Secrets Manager
- Newer service, meant for storing secrets
- Capability to force **rotation of secrets every X day**
- Automate generation of secrets on rotation (uses Lambda)
- Integration with AmazonRDS (MySQL, PostgreSQL,Aurora)
- Secrets are encrypted using KMS
- Mostly meant for RDS integration

### Multi Region Secrets
- Replicate Secrets across multiple AWS regions
- Secrets Manager keeps read replicas in sync with the primary Secret
- Ability to promote a read replica Secret to a standalone secret
- Use cases: multi- region apps , disaster recovery strategies, multi region DB
- ![[Pasted image 20250807183119.png]]
## AWS Certificate Manager - ACM
- Easily provision, manage , and deploy **TLS Certificates**
- Provide in flight encrpytion for websites (HTTPS)
- ![[Pasted image 20250807183403.png]]
- Supports both public and private TLS certs
- Free for public certs
- automatic TLS cert renewal
- Integration with (load TLS certs on)
	- ELB (CLB,ALB,NLB)
	- CloudFront distributions
	- APIs on API Gateway
- Cannot use ACM with EC2 (can't be extracted)

### Requesting Public certs
- List domain names to be included in the cert
	- Fully qualified domain name (FQDN): corp.example.com
	- Wildcard domain: *.example.com 
- Select Validation Method:DNS Validation or Email validation
	- DNS validation is preferred for automation purposes
	- Email validation will send emails to contact addresses in the WHOIS db
	- DNS Validation will leverage a CNAME records to DNS config (Route 53)
- It will take a few hrs to get verified
- The Public Cert will be enrolled for automatic renewal
	- ACM automatically renews ACM generated certs 60 days before expiry

### Importing Public certs
- Options to generate the cert outside of ACM and then import it 
- No automatic renewal, must import a new cert before expiry
- ACM sends daily expiration events starting 45 days prior to expiration
	- The # of days can be configured
	- Events are appearing in EventBridge
- **AWS Config** has a manged rule named `acm-certificate-expiration-check` to check for expiring certs (configurable numbers of days)
- ![[Pasted image 20250807183923.png]]


### Integration with ALB
- ![[Pasted image 20250807184004.png]]

### API Gateway-  Endpoint Types
- Edge-Optimized (default): for global clients
	- Requests are routed through the CloudFront Edge locations (improves latency) 
	- The API Gateway still lives in only one region
- Regional:
	- For Clients within the same region
	- Could manually combine with CloudFront (more control over the caching strategies and the distribution)
- Private:
	- Can only be accessed from your vpc using an interface vpc endpoint (ENI)
	- use resource policy to define access
### Integration with API Gateway
- Create a **Custom Domain Name** in API Gateway
- Edge-Optimized (default) for global clients
	- Requests are routed through the CloudFront edge locations
	- The API Gateway still lives in one region
	- **The TLS cert must be in the same region as CloudFront**
	- Then setup CNAME or (better) A-Alias record in Route 53
	- Always us-east-1
- Regional:
	- For clients within the same region
	- **The TLS Cert must be imported on API gateway, in the same region as the API stage**
	- Then setup CNAME or (better) A-Alias record in Route 53
- ![[Pasted image 20250807184309.png]]

## Web Application Firewall - WAF
- Protect your web app from common web exploits (Layer 7)
- Layer is HTTP (vs Layer 4 is TCP/UDP)
- Deploy on
	- ALB
	- API GW
	- CloudFront
	- AppSync GraphQL API
	- Cognito User Pool
- Define Web ACL (Web Access Control List) Rules:
	- **IP Set: up to 10,000 IP addresses** - use multiple Rules for more IPs
	- HTTP headers, HTTP body, URI strings Protects from common attack - **SQL Injection** and **Cross-Site Scripting**(XSS)
	- Size constraints, geo-match (block countries)
	- **Rate-based rules** (to count occurrences of events) - **for DDoS protection**
- Web ACL are Regional except for cloudfront
- A rule group is a reusable set of rules that you can add to a web ACL

### Fixed IP while using WAF with a LB
- WAF does not support the NLB (Layer 4)
- We can use Global Accelerator for fixed  IP and WAF on the ALB
- ![[Pasted image 20250808201509.png]]

## AWS Shield: protect from DDoS attack
- DDoS: Distributed Denial of Service - many requests at the same time
- AWS Shield Standard:
	- Free service that is activated for every AWS customer
	- Provides protection from attacks such as SYN/UDP Floods, Reflection attacks and other layer 3 / layer 4 attacks
- **AWS Shield Advanced**:
	- Optional DDoS mitigation service ($3000 per month per org)
	- Protect against more sophisticated attack on Ec2, ELB, Cloudfront, Global Accelerator, Route 53
	- 24/7access to AWS DDoS response team (DRP)
	- Protect against higher fees during usage spikes due to DDoS
	- Shield Advanced automatic application layer DDoS mitigation automatically creates, evaluates and deploys AWS WAF rules to mitigate layer 7 attacks

## Firewall Manager
- Manage rules in all accounts of an AWS Org
- Security policy: common set of security rules
	- Waf rules (ALB,API GW, Cloudfront)
	- AWS Shield Advanced (ALB,CLB,NLB, Elastic IP, CloudFront)
	- Security Groups for EC2, ALB, ENI resources in VPC
	- AWS Network Firewall (VPC level)
	- Route 53 Resolver DNS Firewall
	- Policies are created at the region level
- **Rules are applied to new resources as they are created (good for compliance) across all and future accounts in your Organization**

## WAF vs Firewall Manager vs Shield
- **WAF,Shield,Firewall Manager are used together for comprehensive protection**
- Define your Web ACL rules in WAF
- For granular protection of your resources, WAF alone is the correct choice
- If you want to use AWS WAF across accounts, accelerate WAF configuration, automate the protection of new resources, use Firewall Manager with WAF
- Shield Advanced adds additional features on top of WAF, such as dedicated support from the Shield Response Team (SRT) and advanced reporting
- If you're prone to frequent DDoS attacks, consider purchasing Shield Advanced

## Best Practices for DDoS 
### Resiliency Edge Location Mitigation (BP1,BP3)
- ![[Pasted image 20250808202514.png]]
- BP1 - CloudFront
	- Web Application delivery at the edge
	- Protect from DDoS Common Attack (SYN floods, UDP reflection...)
- BP1 - Global Accelerator
	- Access your application from the edge
	- Integration with Shield for DDoS protection
	- Helpful if your backend is not compatible with CloudFront
- BP3 - Route 53
	- Domain Name Resolution at the edge
	- DDoS protection mechanism
###  DDoS Mitigation
- Infrastructure layer defense (BP1, BP3,BP6)
	- Protect Amazon EC2 against high traffic
	- That includes using Global Accelerator, Route 53, CloudFront, Elastic Load Balancing
- EC2 with Auto Scaling (BP7)
	- Helps scale in case of sudden traffic surges including a flash crowd or a DDoS attack
- Elastic Load Balacing (BP6)
	- ELB scales with the traffic increases and will distribute the trafffic to many EC2 instances
### Application Layer Defense
- Detect and filter malicious web requests (BP1,BP2)
	- Cloudfront cache static content and serve it from edge locations, protecting your backend
	- WAF is used on top of CloudFront and ALB to filter and block requests based on request signatures
	- WAF rate-based rules can automatically block the IPs of bad actors
	- Use managed rules on WAF to block attack based on IP reputation, or block anonymous Ips
	- Cloudfront can block specific geographies
- Shield Advanced (BP1,Bp2,Bp6)
	- Shield advanced automatic application layer DDoS mitigation automatically creates eval and deploy WAF rules to mitigate layer 7 attacks

### Attack surgace reduction
- Obfuscating AWS resources (BP1,Bp4,BP6)
	- using cloudfront, API GW, ELB to hide your backend resources
- Security groups and network ACLs (BP5)
	- Use SGs and NACLs to filter traffic based on specific IP at the subnet or ENI level
	- Elastic IP are protected by AWS Shield Advanced
- Protecting API endpoints (BP4)
	- Hide Ec2 , lambda elsewhere
	- Edge-optimized mode, or cloudfront + regional mode (more control for DDoS)
	- WAF + API GW: burst limits , headers filtering, use API keys

## Amazon Guardduty
- Intelligent Threat discovery to protect your AW acc
- Uses ML algo, anomaly detection, 3rd party data
- one click to enable (30 days trial), no need to install software
- Input data includes:
	- **CloudTrail Events Logs**- unusual API calls, unauthorized deployments
		- **CloudTrail Management Events:** create VPC subnet, create trail
		- **CloudTrail S3 Data Events** - get objects, list objects, delete object,...
	- **VPC Flow Logs**: unusual internal traffic, unusual IP address
	- **DNS logs** - compromised EC2 instances sending encoded data within DNS queries
	- **Optional features:** EKS audit logs, RDS & Aurora, EBS,Lambda,S3 data events,...
- Can setup **EventBridge** rules to be notified in case of findings
- EventBridge rules can target Lambda or SNS 
- **Can protect against Cryptocurrency attacks (has a dedicated findings for it)**
- ![[Pasted image 20250808203349.png]]

## Amazon Inspector
- Automated Security Assessments
- For Ec2 instances:
	- Leveraging the **AWS System Manager (SSM) agent**
	- Analyze against **unintended network accessibility**
	- Analyze the **running OS** against **known vulnerabilities**
- **For container images push to Amazon ECR**
	- Assessment of container images as they are pushed
- **For lambda functions**
	- Identifies software vulnerabilities in function code and package dependencies
	- Assessment of functions as they are deployed
- Reporting & integration with AWS security Hub
- Send Findings to EventBridge
- ![[Pasted image 20250808203606.png]]

### What does Inspector evaluate?
- **Remember: only for EC2 instances, Container Images & Lambda functions**
- Continuous scanning of the infra, only when needed
- Package vulnerabilities (EC2, ECR & Lambda) - database of CVE
- Network readability (EC2)
- A risk score is associated with all vulnerabilities for priritization


## Amazon Macie
- fully managed data security and data privacy service that uses **ML and pattern matching to discover and protect your sensitive data in AWS**
- Macie helps identify and alert you to **sensitive data, such as personally identifiable information (PII)**
- ![[Pasted image 20250808203824.png]]
