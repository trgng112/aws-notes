### Elastic Load Balancer
![[Pasted image 20250719202129.png]]

- Why use a load balancer
	- Spread load across multiple downstream instances
	- expose a single point of access (DNS) to app
	- handle failures of downstream services
	- health checks
	- provide SSL termination (HTTPS) for websites
	- enforce stickiness with cookies
	- high availability across zones
	- separate public from private traffic
	- is a managed load balancer
		- aws guarantees it will be working
		- aws take cares of upgrades, maintenance 
		- aws provides a few configs knobs
	- integrated with many AWS services
		- ec2,ecs
		- aws certificate manager, cloudwatch,
		- route 53, waf, global accelerator
	- Health checks
		- Enable the load balancer to know if the instance is available to reply to requests
		- crucial for load balancers
		- check by ports and routes (/health)
		- if the response is not 200, it is unhealthy
	- Types:
		- classic load balancer - CLB - deprecated
		- Application load balancer - ALB 
			- HTTPS,HTTPS,websocket
		- Network load Balancer - NLB
			- TCP TLS UDP
		- Gateway load balancer - GWLB
			- Operated at layer 3 - IP protocol
		- recommended to use the latest version
		- can be set up as internal (private) or external (public) ELBs


### Load balancer security groups
![[Pasted image 20250719232249.png]]

### Application Load balancer - ALB

- Layer 7 - HTTP
- Load balancing to multiple HTTP aplications across machines (target groups)
- load balancing to multiple applications on the same machine (containers)
- support for HTTP/2 and websocket
- support redirects (from HTTP to HTTPS for example)
- routing tables to different target groups
	- routing based on path in URL (example.com/users & example.com/posts)
	- routing based on hostname in URL (one.example.com & other.example.com)
	- routing based on Query string, header (example.com/users?id=123&order=false)
- Great fit for microservices and container based application (docker and amazon ecs)
- has a port mapping feature to redirect to a dynamic port in ECS
- in comparison, we'd need multiple CLB per app
- ![[Pasted image 20250719232654.png]]
- Target groups
	- EC2 instances (can be managed by an Auto Scaling Group) - HTTP
	- ECS tasks( managed by ECS itself) - HTTP
	- Lambda functions - HTTP request is translated into a JSON event
	- IP addresses - must be private IPs
	- ALB can route to multiple target groups
	- Health checks are at the target group level
- ![[Pasted image 20250719232847.png]]
- fixed hostname
- the application servers don't see the IP of the client directly (x the header `x-Forwarded-For)
- We can also get port `X-Forwarded-port` and proto `X-Forwarded-Proto`
- ![[Pasted image 20250719233015.png]]

### Network load balancer - NLB

- Layer4
- Allows:
	- Forward TCP & UDP traffic to your instances
	- Handle millions of request per seconds
	- ultra low latency
- NLB has one static IP per AZ, and supports assigning Elastic IP (helpful for whitelisting specific IP)
- NLB are used for extreme performance, TCP or UDP traffic
- not included in AWS free tier
- ![[Pasted image 20250719234827.png]]
- Target groups:
	- EC2 instances
	- IP addresses - must be private IPs
	- Application load balancer - ALB 
	- Health check support the TCP HTTP and HTTPS protocols
	- ![[Pasted image 20250719234906.png]]
	- 