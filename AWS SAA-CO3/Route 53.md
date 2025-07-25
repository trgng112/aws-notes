## What is a DNS
- Domain Name System which translate the human friendly hostnames into the machine IP addresses
- www.google.com => 172.217.18.36
- Backbone of the internet
- uses hierarchical naming structure
	- .com
	- example.com
	- www.example.com
	- api.example.com
#### Terminologies


- Domain Registrar : Amazon Route 53, GoDaddy, …
- DNS Records: A, AAAA, CNAME, NS, …
- Zone File: contains DNS records
- Name Server : resolves DNS queries (Authoritative or Non-Authoritative)
- Top Level Domain (TLD): .com, .us, .in, .gov, .org, …
- Second Level Domain (SLD): amazon.com, google.com, …
- ![[Pasted image 20250723225422.png]]
- ![[Pasted image 20250723225652.png]]



## Route 53:
- Highly available, scalable, fully managed and Authorative DNS
	- Authoritative = the customer (you) can update the DNS records
- Route 53 is also a Domain Registrar
- Ability to check the health of your resources
- The only AWS service which provided 100% availability SLA
- Why route 53? 53 is a reference to the traditional DNS port
- ![[Pasted image 20250723225836.png]]


#### Records:
- How you want to route traffic for a domain
- Each records contains:
	- Domain / subdomain Name - eg: example.com
	- Record Type - A, AAAA
	- Value - 12.34.56.78
	- Routing Policy - how Route 53 responds to queries
	- TTL - amount of time the record cached at DNS Resolvers
- Route 53 supports the following DNS record types:
	- (must know) A/ AAAA / CName/ NS
	- (advanced) CAA / DS / MX/ NAPTR/ PTR/ SOA/ TXT/ SPF/SRV
- Record Types
	- A - maps a hostname to IPv4
	- AAAA - maps a hostname to IPv6
	- CName - maps a hostname to another hostname
		- The target is a domain name which must have an A or AAAA record
		- Can't create a CNAME record for the top node of a DNS namespace(Zone Apex)
		- Example:  you can't create for example.com, but you  can create for www.example.com
	- NS - Name Servers for the Hosted Zone
		- Control how traffic is routed for a domain

#### Hosted Zones
- A container for records that define how to route traffic to a domain and its subdomains
- Public Hosted Zones - contains records that specify how to route traffic on the Internet (public domain names) application1.mypublicdomain.com
- Private hosted zones: - contain records that specify how you route traffic within one or more VPCs (private domain names) application1.company.internal
- You pay 0.5$ per month per hosted zone
- Public vs Pirivate hosted zones
	- ![[Pasted image 20250723230350.png]]

#### Records TTL - Time to live
- High TTL - 24 hrs:
	- Less traffic on Route 53
	- Possibly outdated records
- Low TTL - 60 sec
	- More traffic on Route 53 - cost more
	-  Records are outdated for less time
	- Easy to change records
- Except for Alias records, TTL is mandatory for each DNS record
- ![[Pasted image 20250723231549.png]]

#### CNAME vs Alias
- AWS Resource (Load Balancer , CLoudfront) expose an AWS hostname:
	- Ib1-1234.us-east-2.elb.amazonaws.com and you want myapp.mydomain.com
- CNAME:
	- Points a hostname to any other hostname (app.mydomain.com => blabla.anything.com)
	- **ONLY FOR NON ROOT DOMAIN (something.mydomain.com)**
- Alias:
	- Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com)
	- **Works for Root Domain and Non Root Domain (aka mydomain.com)**
	- Free of charge
	- Native health check
	- Alias Records
		- Maps a hostname to an AWs resource
		- An extension to DNS functionality
		- Automatically regconizes changes in the resource's IP addresses
		- Unlike CNAME, it can be used for the top node of a DNS namespace (Zone Apex), e.g: example.com
		- Alias Record is always of type A/AAAA for AWS resources (IPv4/ IPv6)
		- Cannot set the TTL
		- ![[Pasted image 20250723232210.png]]
		- Alias Record Targets
			- ELB, Cloudfront distributions, API Gateway, Elastic Beanstalk env, S3 websites,VPC interface Endpoints, Global Accelerator accelerator
			- Route 53 record in the same hosted zone
			- **You cannot set an ALIAS record for an EC2 DNS name**


### Routing Policies
- Defines how Route 53 responds to DNS queries
- Don't get confused by the word "routing"
	- It's not the same as load balancer routing with routes the traffic
	- DNS does not route any traffic, it only responds to the DNS queries
	- Route 53 supports the following routing policies
		- Simple
		- Weighted
		- Failover
		- Latency based
		- Geolocation
		- Multi-value Answer
		- Geoproximity (using Route 53 Traffic flow feature)

#### Simple
- Typically route traffic to a single resource
- Can specify multiple values in the same record
- **If multiple values are returned, a random one is chosen by the client**
- When alias enabled, specify only one AWS resource
- Can't be associated with health checks
- ![[Pasted image 20250725163557.png]]

#### Weighted
- Control the % the requests that go to each specific resource
- Assign each record a relative weight:
	- traffic (%) = weight for a specific record / Sum of all the weights for all records
	- Weights don't need to sum up to 100
- DNS records must have the same name and type
- Can be associated with HGealth checks
- uses case: load balancing between regions, testing new application versions...
- Assign a weight of 0 to a record to stop sending traffic to a resource
- If all records have weight of 0, then all records will be returned equally 
- ![[Pasted image 20250725163959.png]]

#### Latency
- Redirects to the resource that has the least latency close to us
- Super helpful when latency for users is a priority
- Latency is based on traffic between users and AWS region
- Germany users may be directed to the US (if that's the lowest Latency)
- Can be associated with Health Checks (has a failover capability)
- ![[Pasted image 20250725164248.png]]

#### Failover - Active - Passive
- ![[Pasted image 20250725165547.png]]
- Health check one resource and if it fails, route to a secondary one - disaster recovery
#### Geolocation
- Different from the Latency-based
- This routing is based on the user location
- Specify location by Continent, Country, by US state (if there's overlapping, most precise location selected)
- Should create a **Default** record (in case there's no match on location)
- Use cases: website localization, restrict content distribution, load balancing,...
- Can be associated with Health Checks
- ![[Pasted image 20250725165927.png]]

#### Geoproximity
- Route traffic to your resources based on the grographic location of users and resources
- Ability to **shift more traffic to resource based** on the defined Bias
- To change the size of the geographic region, specify bias values:
	- To expand (1 to 99) - more traffic to the resource
	- To shrink (-1 to -99) - less traffic to the resource
- Resources can be:
	- AWS resources (specify AWS region)
	- Non-AWS resources (specify Latitude and Longitude)
- You must use Route 53 Traffic Flow (advanced) to use this feature
- ![[Pasted image 20250725170424.png]]
- ![[Pasted image 20250725170436.png]]
#### IP address
- Routing is based on client's IP addresses
- You provide a list of CIDRs for your clients and the corresponding endpoints/locations
- Use cases: optimize performance, reduce network costs
- Example: route end users from a particular ISP to a specific endpoint
- ![[Pasted image 20250725172056.png]]

#### Multi - value
- use when routing traffic to multiple resources
- Route 53 return multiple values / resources
- Can be associated with Health Checks (return only values for healthy resources)
- Up to 8 healthy records are returned for each Multi value query
- Multi value is not a substitute for having an ELB
![[Pasted image 20250725172226.png]]
### Health checks
- HTTP Health checks are only for public resources
- Health check => Automated DNS Failover:
	- Health checks that monitor an endpoint (application,server, other AWS resource)
	- Health checks that monitor other health checks (Calculated Health Checks)
	- Health checks that monitor CloudWatch Alarms (full control) - throttles of DynamoDB, alarm on RDS, custom metrics (helpful for private resources)
-  Health checks are integrated with CW metrics
- ![[Pasted image 20250725164636.png]]
#### Monitor an Endpoint
- About 15 global health checkers will check the endpoint health
	- Healthy / Unhealthy threshold - 3 default
	- Interval - 30 sec (can set to 10 sec - higher cost)
	- Supported protocol : HTTP, HTTPS , TCP
	- if > 18% of health checkers report the endpoint is healthy, Route 53 considers it **Healthy**, otherwise it's **Unhealthy**
	- Ability to choose which locations you want Route 53 to use
- Health checks pass only when endpoint response with the 2xx and 3xx status codes
- Health checks can be setup to pass/fail based on the text in the first **5120 bytes** of the response
- Configure your router/firewall to allow incoming requests from Route 53 Health Checkers
- ![[Pasted image 20250725164843.png]]

#### Calculated Health Check
- Combine the results of multiple HC into a single HC
- You can use OR, AND, NOT
- Can monitor up to 256 Child HCs
- Specify how many of the health checks need to pass to make the parent pass
- usage: perform maintenance to your website without causing all health checks to fail
- ![[Pasted image 20250725165027.png]]

#### Private Hosted Zones
- Route 53 HCs are outside the VPC
- They can't access private endpoints (private VPC or on-premises resource)
- You can create a Cloudwatch metric and associate a CloudWatch alarm, then create a Health check that checks the alarm itself
- ![[Pasted image 20250725165140.png]]


### Domain Registrar vs DNS Service
- You buy or register your domain name with a Domain Registrar typically by paying annual charges (GoDaddy, Amazon Registrar)
- The Domain Registrar usually provides you with a DNS service to manage your DNS records
- But you can use another DNS service to manage your DNS records
- Example: purchase the domain from GoDaddy and use Route 53 to manage your DNS records
- ![[Pasted image 20250725172533.png]]
- ![[Pasted image 20250725172608.png]]
- **If you buy your domain on a 3rd party registrar, you can still use Route 53 as the DNS Service Provider**
	- Create a Public Hosted Zone in Route 53
	- Update NS records on 3rd party website to use Route 53 Name Servers
- **Domain Registrar** != DNS Service
- But every Domain Registrar usually comes with some DNS features