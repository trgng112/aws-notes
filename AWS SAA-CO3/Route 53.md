### What is a DNS
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



### Route 53:
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
			- 