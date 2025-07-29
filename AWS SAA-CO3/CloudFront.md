- Content Delivery Network (CDN)
- Global distribution
- Improved read performance, content is cached at the edge
- Improves user experience
- 216 Point of Presence globally (Edge location)
- **DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall**
-  ![[Pasted image 20250728153302.png]]


### Origins
- S3 Bucket
	- For distributing files and caching them at the edge
	- For uploading files to S3 through CloudFront
	- Secured using Originb Access Control (OAC)
- VPC Origin
	- For applications hosted in VPC private subnets
	- Application Load Balancer / Network Load Balancer/ EC2 Instances
- Custom Origin (HTTP)
	- S3 website (must first enable the bucket as a static s3 websites)
	-  any public http backend
- ![[Pasted image 20250728153606.png]]
- ![[Pasted image 20250728153646.png]]


#### CloudFront vs S3 Cross Region Replication
- CloudFront:
	- Global edge network
	- Files are cached for a TTL (maybe a day)
	- **Great for static content that must be available everywhere**
- S3 Cross Region Replication:
	- Must be setup for each region you want replication to happen
	- Files are updated in near real-time
	- Read-only
	- **Great for dynamic content that needs to be available at low-latency in few regions*

#### CloudFront - ALB or EC2 as an origin using VPC Origins
- Allow you to deliver content from your application s hosted in your VPC private subnets (no need to expose them on the internet)
- Deliver traffic to **private**:
	- Application Load Balancer
	- Network Load Balancer
	- EC2 instances
	- ![[Pasted image 20250728154648.png]]
- Using **Public Network**
	- ![[Pasted image 20250728154730.png]]

### Geo Restriction
- You can restrict who can access your distribution
	- Allowlist: allow your users to access your content only it they're in one of the countries on a list of approved countries
	- Blocklist: Prevent your users from accessing your content if they're in one of the countries on a list of banned countries.
- The "country" is determined using a 3rd party Geo-IP database
- Use case: Copyright Laws to control access to content

### Price Classes
- CloudFront Edge locations are all around the world
- The cost of data out per edge location varies
- ![[Pasted image 20250728155000.png]]
- You can reduce the number of edge locations for cost reduction
- Three price classes:
	- Price Class All: all regions - best performance
	- Price Class 200: most regions, but excludes the most expensive regions
	- Price Class 100: Only the least expensive regions
	- ![[Pasted image 20250728155129.png]]
	- .![[Pasted image 20250728155137.png]]

### Cache Invalidations
- In case you update the backend origin, CloudFront doesn't know about it and will only get the refreshed content after the TTL has expired
-  However you can force an entire or partial cache refresh (thus bypassing the TTL) by perofrming a CloudFront Invalidation
- You can invalidate all files (*) or a special path (/images/*)
- ![[Pasted image 20250728155354.png]]

### AWS Global Accelerator
- You have deployed an application and have global users who want to access it directly
- They go over the public internet , which can add a lot of latency due to many hops
- We wish to go as fast as possible through AWS Network to minimize latency
- ![[Pasted image 20250728155519.png]]


#### Unicast IP vs Anycast IP
- Unicast IP: one server holds one IP address
- AnycastIP : all servers hold the same IP address and the client is routed to the nearest one
- ![[Pasted image 20250728155652.png]]

#### Global Accelerator
- Leverage the AWS internal network to route your application
- 2 Anycast IP are created for your application
- The Anycast IP send traffic directly to Edge Locations
- The Edge Llocations send the traffic to your application
- ![[Pasted image 20250728155738.png]]
- Works with Elastic IP, EC2 instances, ALB,NLB,public or private
- Consistent Performance
	- Intelligent routing to lowest latency and fast regional failover
	- No issue with client cache (Because the IP doesn't change)
	- Internal AWS network
- Health checks
	- Global Accelerator performs a health check of your applications
	- Helps make your application global (failover less than 1 minute for unhealthy)
	- Great for disaster discovery (thanks to the health checks)
- Security
	- Only 2 external IP need to be whitelisted
	- DDoS protection thanks to AWS shield

### Global Accelerator vs CloudFront
- They both use the AWS Global network and its edge locations around the world
- Both service integrate with AWS Shield for DDoS protection
- Cloudfront:
	- Improves performance for both cacheable content (such as images and videos)
	- Dynamic content (such as API acceleration and dynamic site delivery)
	- Content is served at the edge
- Global Accelerator
	- Improves performance for a wide range of applications over TCP or UDP
	- Proxying packets at the edge to applications running in one or more AWS Regions.
	- Good fit for non HTTP use cases, such as gaming (UDP),IoT (MQTT) or Voice over IP 
	- Good for HTTP use cases that require static IP addresses
	- Good for HTTP use cases that required deterministic,fast regional failover