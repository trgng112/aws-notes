- ![[Pasted image 20250809103633.png]]

## CIDR - IPv4
- Classless Inter - Domain Routing - a method for allocating IP addresses
- Used in SGs rules and AWS networking in general
- ![[Pasted image 20250809103748.png]]
- They help to define an IP address range:
	- We've seen WW.XX.YY.ZZ/32 => one IP
	- We've seen 0.0.0/0 => all Ips
	- we can define: 192.168.0.0/26 => 192.168.0.0 - 192.168.0.63 (64 ip addresses)
- CIDR consists of two components
- Base IP
	- Represent an IP contained in the range (XX.XX.XX.XX)
	- Example: 10.0.0.0,192.168.0.0
- Subnet Mask
	- Defines how many bits can change in the IP
	- example /0,/24,/32
	- Can take two forms:
		- /8=> 255.0.0.0
		- /16 => 255.255.0.0
		- /24 => 255.255.255.0
		- /32 => 255.255.255.255
### Subnet Mask
- The subnet mask basically allows part of the underlying IP to get additional next values from the base IP
- ![[Pasted image 20250809104055.png]]
- ![[Pasted image 20250809104127.png]]

### Public vs Private IP (IPv4)
- The internet assigned Numbers Authority (IANA) established certain blocks of IPv4 addresses for the use of private (LAN) and public (Internet) addresses
- **Private IP** can only allow certain values:
	- 10.0.0.0 - 10.255.255.255 (10.0.0.0/8) => in big networks
	- 172.16.0.0 - 172.31.255.255 (172.16.0.0/12) => AWS default VPC in that range
	- 192.168.0.0 - 192.168.255.255 (192.168.0.0/16) => home networks
- All the rest of the IP addresses on the internet are public

## Default VPC
- All new AWS accounts have a default VPC
- New EC2 instances are launched into the default VPC if no subnet is specified
- Default VPC has Internet connectivity and ALL Ec2 instances inside it have public ipv4 addresses
- We also get a public and a private IPv4 DNS names

## VPC in AWS - IPv4
- VPC = Virtual Private Cloud
- You can have multiple VPCs in an AWS region (max 5 per region - soft limit)
- Max CIDR per VPC is 5, for each CIDR:
	- Min size is /28 (16 ip addresses)
	- Max is /16 (65536 ip addresses)
- Because VPC is private, only the Private IPv4 ranges are allowed:
	- 10.0.0.0 - 10.255.255.255 (10.0.0.0/8)
	- 172.16.0.0 - 172.31.255.255(172.16.0.0/12)
	- 192.168.0.0 - 192.168.255.255 (192.168.0.0/16)
- **Your VPC CIDR should NOT overlap with your other networks (eg corporate**


## Subnet
- ![[Pasted image 20250809105052.png]]

- AWS r**eserves 5 IP addresses (first 4 and last 1 )** in each subnet
- These 5 ip addresses are not available for use and cant be assigned to an EC2 instance
- example: if CIDR block 10.0.0.0/'24 , then reserved IP addresses are:
	- 10.0.0.0 - Network address
	- 10.0.0.1 - reserve by AWS for the VPC router
	- 10.0.0.2 - reserved by AWS for mapping to amazon provided DNS
	- 10.0.0.3 - reserved by AWS for future use
	- 10.0.0.255 - network broadcast address. AWS does not support broadcast in a vpc, therefore the address is reserved
- **Exam tip**: if you need 29 ip addresses for Ec2 instance:
	- You cant choose a subnet of size /27 (32 ip addresses, 32-5 = 27 <29)
	- You need to choose a subnet of size /26 (64 ip addresses , 64-5 = 59>29)

## Internet Gateway (IGW)
- Allow resources (EC2 instances) in a VPC connect to the internet
- it scales horizontally and is highly available and redundant
- must be created separately from a VPC
- one vpc can only be attached to one IGW and vice versa
- IGWs on their own do not allow internet access
- Route tables must also be edited
- ![[Pasted image 20250809105623.png]]

c## Bastion Hosts
- ![[Pasted image 20250809110046.png]]
- We can use a bastion host to SSH into our private Ec2 instances
- the bastion is in the public subnet which is connected to all other private subnets
- **Bastion Host security group must allow** in bound from the internet on port 22 from restricted CIDR, for example **the public CIDR** of your corporation
- SG of the Ec2 Instances must allow the SG of the Bastion Host, or the Private IP of the Bastion Host

## NAT instances
- NAT = Network Address Translation
- Allows EC2 instances in private subnets to connect to the other internet
- Must be launched in a public subnet
- must disable EC2 setting : **Source/destination Check**
- Must have Elastic IP attached to it
- Route Tables must be configured to route traffic from private subnets to the NAT Instance
- ![[Pasted image 20250809151216.png]]
- ![[Pasted image 20250809151230.png]]
- Not highly available / resilient setup out of the box
	- You need to create an ASG in multi AZ + resilient user-data script
- Internet traffic bandwidth depends on EC2 instance type
- You must manage SGs & rules;
	- Inbound:
		- Allow HTTP/HTTPS traffic coming from Private Subnets
		- Allow SSH from your home network (access is provided through Internet Gateway)
	- Outbound:
		- Allow HTTP/HTTPS traffic to the Internet
## NAT gateway
- AWS managed NAT, higher bandwidth , high availability,no administration
- Pay per hour for usage and bandwidth
- NATGW is created in a specific AZ, uses an Elastic IP
- Can't be used by EC2 instance in the same subnet (only from other subnets)
- Requires an IGW (Private Subnet => NATGW => IGW)
- 5GBPS of bandwidth with automatic scaling up to 100Gbps
- No SGs to manage/required
- ![[Pasted image 20250809151746.png]]

### NAT GW with High availability
- NAT GW is resilient within a single AZ
- Must create multiple NAT GWs in multiple AZs for fault-tolerance
- There is no cross-AZ failover needed because if an AZ goes down it doesn't need NAT
- ![[Pasted image 20250809151843.png]]
- ![[Pasted image 20250809151856.png]]

## NACL & Security Groups
- ![[Pasted image 20250809152251.png]]

### Network Access Control List (NACL)
- NACL are like firewall which control traffic from and to subnets
- **One NACL per subnet**, new subnets are assigned the **Default NACL**
- You define **NACL Rules:**
	- Rules have a number (1-32766), higher precedence with a lower number
	- first rule match will drive the decision
	- Example: if you define #100 ALLOW 10.0.0.10/32 and #200 DENY 10.0.0.10/32, the IP address will be allowed because 100 has a higher precedence over 200
	- The last rule is an asterisk * and denies a request in case of no rule match
	- AWS recommends adding rules by increment of 100
- Newly created NACLs will deny everything
- NACL are a great way of blocking a specific IP address at the subnet level

#### Default NACL
- Accepts everything inbound/outbound with the subnets its associated with
- Do NOT modify the Default NACL, instead create custom NACLs
- ![[Pasted image 20250809152540.png]]

### Ephemeral Ports
- For any two endpoints to establish a connection, the must use ports
- Clients connect to a **defined port**, and expect a response on an **ephemeral port**
- Different Operating Systems use different port ranges, examples:
	- IANA & MS Windows 10 => 49152 - 65535
	- Many Linux Kernels => 32768 => 60999
- ![[Pasted image 20250809152736.png]]
### NACL with Ephemeral Ports
-  ![[Pasted image 20250809152838.png]]
- ![[Pasted image 20250809152852.png]]
### Security Group vs NACls
- ![[Pasted image 20250809152903.png]]


## VPC Peering
- Privately connect two VPCs using AWS network
- Make them behave as if they were in the same network
- Must not have overlapping CIDRs
- VPC Peering connection is **NOT transitive** (must be established for each VPC that need to communicate with one another)
- **You must update route tables in each VPC's subnets to ensure Ec2 instances can communicate with each other**
-  ![[Pasted image 20250809153404.png]]
- You can create VPC Peering connection between VPCs in **different AWS accounts/regions**
- You can reference a security group in a peered VPC (works cross account - same region)
- ![[Pasted image 20250809153501.png]]
- ![[Pasted image 20250809153514.png]]

## VPC Endpoints
- ![[Pasted image 20250810185107.png]]
- Every AWS service is public exposed (public URL)
- VPC Endpoints (AWS PrivateLink) allows you to connect to AWS services using a **private network** instead of using the public internet
- They're redundant and scale horizontally
- They remove the need of IGW, NATGW,... to access AWS services
- In case of issues:
	- Check DNS Setting Resolution in your VPC
	- Check Route Tables
### Types of Endpoints
- Interface Endpoints (powered by Private Link)
	- Provisions an ENI (private IP address) as an entry point (must attach a SG)
	- Supports most AWS services
	- $per hour + $ per GB of data processed
	- ![[Pasted image 20250810185328.png]]
- Gateway Endpoints:
	- Provisions a gateway and must be used as a target in a route table (dues not use SGs)
	- Supports both S3 and DynamoDB
	- Free
	- ![[Pasted image 20250810185416.png]]

### Gateway or Interface Endpoint for S3
- Gateway is most likely going to be preferred all the time at the exam
- Cost: free  for Gateway, $ for interface
- Interface Endpoint is preferred access is required from on-premise (site to site vpn or direct connect), a different VPC or a different region
- ![[Pasted image 20250810185532.png]]

## VPC Flow Logs
- Capture information about IP traffic going into your interfaces
	- VPC FLow Logs
	- Subnet Flow Logs
	- Elastic Network Interface (ENI) Flow logs
- Helps to monitor & troubleshoot connectivity issues
- Flow logs data can go to S3, CW Logs, and Kinesis Data Firehose
- Captures network information from AWS managed interfaces too: ELB,RDS,ElastiCache,Redshift,workspaces,NATGW,Transit GW,...

### Syntax
- ![[Pasted image 20250810190035.png]]

### Troubleshoot SG & NACL issues
- Look at the **Action** field
- ![[Pasted image 20250810190124.png]]

### Architecture
- ![[Pasted image 20250810190159.png]]


## Site to Site VPN
- ![[Pasted image 20250810190737.png]]
- Virtual Private Gateway (VGW)
	- VPN concentrator on the AWS side of the VPN connection
	- VGW is created and attached to the VPC from which you want to create the Site to Site VPN connection
	- Possibility to customize the ASN (Autonomous System Number)
- Customer Gateway (CGW)
	- Software application or physical device on customer side of the VPN connection

### VPN Connections
- Customer Gateway device (on-premises)
	- What IP address to use?
		- Public Internet routable IP address for your Customer Gateway device
		- if it's behind a NAT device that's enabled for NAT traversal (NAT-T), use the public IP address of the NAT device
- **Important step**: enable **Route Propagation** for the Virtual Private Gateway in the route table that is associated with your subnets
- If you need to ping your Ec2 instances from on premises, make sure you add the ICMP protocol on the inbound of your SGs

### AWS VPN Cloudhub
- Provide secure communication between multiple sites if you have multiple vpn connections
- low-cost hub-and-spoke model for primary or secondary network connectivity between different locations (VPN only)
- It's a VPN connection so it goes over the public Internet
- To set it up, connect multiple VPN connections on the same VGW, setup dynamic routing and configure route tables
- ![[Pasted image 20250810191143.png]]

## Direct Connect and Direct Connect Gateway
- Provides a dedicated **private** connection from a remote network to your VPC
- Dedicated connection must be setup between your DC and AWS Direct Connect locations
- You need to setup a Virtual Private GW on your VPC
- Access public resources (S3) and private (EC2) on the same connection
- Use cases:
	- Increase bandwidth throughput - working with large data sets - lower cost
	- More consistent network experience - applications using real-time data feeds
	- Hybrid envs (on prem+ cloud)
- supports both IPv4 and IPv6
- ![[Pasted image 20250810191504.png]]
### Direct Connect Gateway
- If you want to setup a Direct Connect to one or more VPC in many different regions (same account), you must use a Direct Connect Gateway
- ![[Pasted image 20250810191547.png]]

### Connection Types
- Dedicated Connections : 1Gbps, 10Gbps and 100Gbps cpaacity
	- Physical ethernet port dedicated to a customer
	- Request made to AWS first, then completed by AWS direct connect partners
- Hosted Connections: 50 Mbps, 500 Mbps, to 10 Gbps
	- Connection requests are made via AWS Direct Connect Partners
	- Capacity can be **added or removed on demand**
	- 1, 2,5, 10 Gbps available at select AWS Direct Connect Partners
- Lead times are often longer than 1 month to establish a new connection

### Encryption
- Data in transit is not encrypted but is private
- Direct Connect + VPN provides an IPsec-encrypted private connection
- Good for and extra level of security, but slightly more complex to put in place
- ![[Pasted image 20250810191819.png]]

### Resiliency
![[Pasted image 20250810191910.png]]


### Site to Site VPN connection as backup
- In case Direct Connect fails, you can set up a backup Direct Connect connection (expensive), or a Site to Site VPN connection
- ![[Pasted image 20250810192009.png]]

## Transit Gateway
- ![[Pasted image 20250810193009.png]]
- For having transitive peering between thousands of VPC and on premises, hub-and-spoke (star) connection
- Regional resource , can work cross region
- share cross-account, using Resource Access Manager (RAM)
- You can peer Transit Gateways across regions
- Route Tables: limit which VPC can talk with other VPC
- Works with Direct connect GW, VPN connections
- Supports **IP Multicast** (not supported by any other AWS services)
- ![[Pasted image 20250810193207.png]]

### Site to site VPN ECMP
- **ECMP = Equal - cost multi-path routing**
- Routing strategy to allow to forward a packet over multiple best path
- Use case: create multiple site-to-site VPN connections **to increase the bandwidth of your connection to AWS**
- ![[Pasted image 20250810193324.png]]
#### Throughput with ECMP
![[Pasted image 20250810193357.png]]
### Share Direct Connect between multiple accounts
- ![[Pasted image 20250810193417.png]]


## VPC Traffic Mirroring
 - Allows you to capture and inspect network traffic in your VPC
 - Route the traffic to security appliances that you manage
 - Capture the traffic
	 - **From (source)** - ENIs
	 - **To (targets)**- an ENI or a NLB
 - Capture all packets or capture the packets of your interest (optionally , truncate packets)
 - Source and target can be in the same VPC or different VPC (VPC Peering)
 - Use cases: content inspection, threat monitoring, troubleshooting
 - ![[Pasted image 20250810193616.png]]

## IPv6 for VPC
### IPv6
- IPv4 designed to provide 4.3 Billion addresses (they'll be exhuasted soon)
- IPv6 is the successor of IPv4
- IPv6 is designed to provide 3.4 x 10^38 unique IP address
- **every IPv6 address in AWS is public** and Internet-routable (no private range)
- Format => x.x.x.x.x.x.x.x (x is hexadecimal,range can be from 0000 to ffff)
- examples:
	![[Pasted image 20250810193754.png]]

### IPV6 in VPC
- **IPv4 cannot be disabled for your VPC and subnets**
- You can enable IP v6 (public IP addresses) to operate in dual-stack mode
- Your EC2 instances will get at least a private internal IPv4 and a public IPv6
- They can communicate using either IPv4 or IPv6 to the internet through an Internet Gateway
- ![[Pasted image 20250810193928.png]]

### IPV4 Troubleshooting
- **IPv4 cannot be disabled for your VPC and subnets**
- So, if you cannot launch an EC2 instance in your subnet
	- It's not because it cannot acquire an IPv6 (the space is very large)
	- It's because there are no available IPv4 in your subnet
- **Solution**: create a new IPv4 CIDR in your subnet
- ![[Pasted image 20250810194046.png]]

## Egress Only Internet Gateway
- **Used for IPv6 only**
- (similar to a NAT Gateway but for IPv6)
- Allows instances in your VPC outbound connections over IPv6 while preventing the internet to initiate an IPv6 connection to your instances
- **You must update the Route Tables**
- ![[Pasted image 20250810194338.png]]
- ![[Pasted image 20250810194444.png]]


## Networking cost in AWS per GB - simplified
- Use Private IP instead of Public IP for good savings and better network performance
- Use same AZ for maximum savings (at the cost of high availability)
- ![[Pasted image 20250810195406.png]]

### Minimizing Egress traffic network cost
- Egress traffic : outbound traffic (from AWs to outside)
- Ingress traffic: inbound traffic - from outside to AWS (typically free)
- Try to keep as much internet traffic within AWS to minimize costs
- **Direct Connect location that are co-located in the same AWS region result in lower cost for egress network**
- ![[Pasted image 20250810195624.png]]
### S3 Data Transfer Pricing - Analysis for USA
- S3 Ingress : free
- S3 to Internet : $0.09 per GB
- S3 Transfer Acceleration:
	- Faster transfer times (50 to 500% better)
	- Additional cost on top of Data Transfer Pricing: +$ 0.04 to $0.08 per GB
- S3 to CloudFront: $0.00 per GB
- CloudFront to Internet: $0.085 per GB (slightly cheaper than S3)
	- Caching capability (lower latency)
	- Reduce costs associated with S3 Requests Pricing (7x cheaper with CloudFront)
- S3 Cross Region Replication : $0.02 per GB
- ![[Pasted image 20250810195837.png]]

### NAT Gateway vs Gateway VPC Endpoint
- ![[Pasted image 20250810195942.png]]

## Network Protection on AWS
- **To protect network on AWS, we've seen** :
	- Network Access Control Lists (NACLs)
	- Amazon VPC security groups
	- AWS WAF (protect against malicious requests)
	- AWS Shield & AWS Shield Advanced
	- AWS Firewall Manager (to manage them across accounts)
- But what if we want to protect in a sophisticated way our entire VPC?
### AWS Network Firewall
 - Protect your entire VPC
 - From Layer 3 to Layer 7 protection
 - Any direction: you can inspect:
	 - VPC to VPC traffic
	 - Outbound to internet
	 - Inbound from internet
	 - To/From Direct Connect & Site-To-Site VPN
 - Internally, the AWS Network Firewall uses the AWS Gateway Load Balancer
 - Rules can be centrally managed cross-account by AWS Firewall Manager to apply to many VPCs
 - ![[Pasted image 20250810200222.png]]
#### Firewall - Fine Grained Controls
- Supports 1000s of rules
	- IP & port - example: 10,000s of IPs filtering
	- Protocol - example: block the SMB protocol for outbound communications
	- Stateful domain list rule groups: only allow outbound traffic to *.mycorp.com* or third-party software repo
	- General pattern matching using reges
- **Traffic filtering: Allow,drop,or alert for the traffic that matches the rules**
- **Active flow inspection** to protect against network threats with intrusion - prevention capabilities (like Gateway LB , but all manged by AWS)
- Send logs of rule matches to S3, CW logs, Kinesis Data Firehose
- 
## VPC Summary
- **CIDR** - IP range
- **VPC** - Virtual Private Cloud => we define a list of IPv4 and IPv6 CIDR
- **Subnets** - tied to an AZ, we define a CIDR
- **Internet Gateway** - at the VPC level, provide IPv4, IPv6 internet access 
- **Route Tables** - must be edited to add routes from subnets to the IGW, VPC peering connections, VPC Endpoints
- **Bastion Host** - public EC2 instance to SSH into, that has SSH connectivity to EC2 instances in private subnets
- **NAT instances** - gives internet access to EC2 instances in private subnets. Old, must be setup in a public subnet, disable Source /  Destination check flag
- **NAT Gateway** - managed by AWS, provides scalable Internet access to private EC2 instances, when the target is an IPv4 address
- **NACL** - stateless, subnet rules for inbound and outbound, fon't forget Ephemeral Ports
- **Security Groups** - stateful, operate at the Ec2 instance level
- **VPC Peering** - connect two VPCs with non overlapping CIDR, non transitive
- **VPC Endpoints**- provide private access to AWS Services (S3, dynamoDB,cloudformation,SSM) within a VPC
- **VPC Flow Logs** - can be setup at the VPC / Subnet/ ENI Level, for ACCEPT and REJECT traffic, helps identifying attacks, analyze using Athena or CW logs insights
- **Site-to-Site VPN** - setup a Customer GW on DC, a Virrtual Private Gateway on VPC, and site to site VPN over public Internet
- **AWS VPN Cloudhub** - hub-and-spoke VPN model to connect your sites
- **Direct Connect** - setup a Virtual Private GW on VPC, and establish a direct private connection to an AWS DC Location
- **Direct Connect Gateway** - setup a Direct Connect to many VPCs in a different AWS regions
- **AWS PrivateLink / VPC Endponit Services**
	- Connect services privately from your service VPC to customers VPC
	- Doesn't need VPC Peering, public Internet, NAT Gateway, Route Tables
	- Must be used with NLB & ENI
- **ClassicLink** - connect EC2 - Classic EC2 instances privately to your VPC
- **Transit Gateway** - Transitive peering connections for VPC, VPN & DX
- **Traffic Mirroring** - copy network traffic from ENIs for further analysis
- **Egress-only Internet Gateway** - Like a NAT GW, but for IPv6
