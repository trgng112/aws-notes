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
- Regional:
	- For clients within the same region
	- **The TLS Cert must be imported on API gateway, in the same region as the API stage**
	- Then setup CNAME or (better) A-Alias record in Route 53
- ![[Pasted image 20250807184309.png]]