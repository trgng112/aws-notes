# ELB - Elastic Load Balancer

## Overview

### What is a Load Balancer?
Load balancers distribute incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses.

![[Pasted image 20250719202129.png]]

### Why Use a Load Balancer?
- **Traffic Distribution**: Spread load across multiple downstream instances
- **Single Access Point**: Expose a single point of access (DNS) to your application
- **Fault Tolerance**: Handle failures of downstream services
- **Health Monitoring**: Perform health checks on instances
- **SSL Termination**: Provide SSL termination (HTTPS) for websites
- **Session Management**: Enforce stickiness with cookies
- **High Availability**: Enable high availability across zones
- **Traffic Separation**: Separate public from private traffic

### AWS Managed Service Benefits
- **Reliability**: AWS guarantees it will be working
- **Maintenance**: AWS takes care of upgrades and maintenance
- **Configuration**: AWS provides configuration options
- **Integration**: Integrated with many AWS services (EC2, ECS, ACM, CloudWatch, Route 53, WAF, Global Accelerator)

## Health Checks

### How Health Checks Work
- Enable the load balancer to know if instances are available to reply to requests
- Crucial for load balancer functionality
- Check by ports and routes (e.g., `/health`)
- If the response is not 200 (OK), the instance is marked as unhealthy

## Load Balancer Types

### Overview of Types
- **Classic Load Balancer (CLB)** - *Deprecated*
- **Application Load Balancer (ALB)** - HTTP, HTTPS, WebSocket
- **Network Load Balancer (NLB)** - TCP, TLS, UDP
- **Gateway Load Balancer (GWLB)** - Operates at Layer 3 (IP protocol)

**Recommendation**: Use the latest versions for better features and performance

**Deployment Options**: Can be set up as internal (private) or external (public) ELBs

## Security Groups for Load Balancers

![[Pasted image 20250719232249.png]]

## Application Load Balancer (ALB)

### Overview
- **Layer 7**: Operates at HTTP level
- **Multi-Application Support**: Load balancing to multiple HTTP applications across machines (target groups)
- **Container Support**: Load balancing to multiple applications on the same machine (containers)
- **Protocol Support**: HTTP/2 and WebSocket
- **Redirects**: Support redirects (e.g., HTTP to HTTPS)

### Routing Capabilities
- **Path-based routing**: Route based on path in URL (`example.com/users` & `example.com/posts`)
- **Hostname-based routing**: Route based on hostname (`one.example.com` & `other.example.com`)
- **Query string/Header routing**: Route based on query string or headers (`example.com/users?id=123&order=false`)

### Use Cases
- **Microservices**: Great fit for microservices and container-based applications
- **Docker & ECS**: Works well with Docker and Amazon ECS
- **Dynamic Ports**: Has port mapping feature to redirect to dynamic ports in ECS
- **Multiple Applications**: In comparison, you'd need multiple CLBs per application

![[Pasted image 20250719232654.png]]

### Target Groups
- **EC2 instances**: Can be managed by an Auto Scaling Group (HTTP)
- **ECS tasks**: Managed by ECS itself (HTTP)
- **Lambda functions**: HTTP request is translated into a JSON event
- **IP addresses**: Must be private IPs
- **Multi-Target Support**: ALB can route to multiple target groups
- **Health Checks**: Performed at the target group level

![[Pasted image 20250719232847.png]]

### Client Connection Details
- **Fixed hostname**: ALB provides a fixed hostname
- **Client IP**: Application servers don't see the client IP directly
- **Headers**: Use `X-Forwarded-For` header to get client IP
- **Additional Headers**: Can also get port (`X-Forwarded-Port`) and protocol (`X-Forwarded-Proto`)

![[Pasted image 20250719233015.png]]

## Network Load Balancer (NLB)

### Overview
- **Layer 4**: Operates at transport layer
- **High Performance**: Handle millions of requests per second
- **Ultra Low Latency**: Optimized for performance
- **Static IP**: One static IP per AZ, supports assigning Elastic IP
- **Use Cases**: Extreme performance requirements, TCP or UDP traffic
- **Cost**: Not included in AWS free tier

![[Pasted image 20250719234827.png]]

### Target Groups
- **EC2 instances**
- **IP addresses**: Must be private IPs
- **Application Load Balancer**: Can target ALB for advanced routing
- **Health Checks**: Support TCP, HTTP, and HTTPS protocols

![[Pasted image 20250719234906.png]]

## Gateway Load Balancer (GWLB)

### Overview
- **Purpose**: Deploy, scale, and manage a fleet of 3rd party network virtual appliances in AWS
- **Use Cases**: Firewalls, intrusion detection and prevention systems, payload manipulation
- **Layer 3**: Operates at network layer (IP packets)
- **Protocol**: Uses GENEVE protocol on port 6081

### Functions
- **Transparent Network Gateway**: Single entry/exit point for all traffic
- **Load Balancer**: Distributes traffic to your virtual appliances

![[Pasted image 20250720120936.png]]

### Target Groups
- **EC2 instances**
- **IP addresses**: Must be private IPs

![[Pasted image 20250720121031.png]]

## Advanced Features

### Sticky Sessions (Session Affinity)
- **Purpose**: Ensure the same client is always redirected to the same instance
- **Compatibility**: Works for CLB, ALB, NLB
- **Mechanism**: Uses cookies with set expiration dates
- **Use Case**: Prevent users from losing session data

![[Pasted image 20250720121154.png]]

#### Cookie Types

##### Application-Based Cookies
**Custom Cookie**:
- Generated by the target application
- Can include any custom attributes required by the application
- Cookie name must be specified individually for each target group
- Cannot use reserved names: `AWSALB`, `AWSALBAPP`, or `AWSALBTG`

**Application Cookie**:
- Generated by the load balancer
- Cookie name is `AWSALBAPP`

##### Duration-Based Cookies
- Generated by the load balancer
- Cookie name is `AWSALB` for ALB, `AWSELB` for CLB

### Cross-Zone Load Balancing
- **Function**: Each load balancer instance distributes evenly across all registered instances in all AZs

![[Pasted image 20250720121901.png]]

#### Configuration by Load Balancer Type

**Application Load Balancer (ALB)**:
- Enabled by default (can be disabled at target group level)
- No charges for inter-AZ data transfer

**Network Load Balancer (NLB) and Gateway Load Balancer (GWLB)**:
- Disabled by default
- Charges apply for inter-AZ data transfer if enabled

**Classic Load Balancer (CLB)**:
- Disabled by default
- No charges for inter-AZ data transfer

## SSL/TLS Configuration

### Overview
- **Purpose**: SSL Certificate allows traffic encryption between clients and load balancer
- **SSL vs TLS**: SSL (Secure Sockets Layer) is older; TLS (Transport Layer Security) is current standard
- **Certificate Authorities**: Public SSL certificates issued by CAs (Comodo, Symantec, GoDaddy, GlobalSign)
- **Expiration**: Certificates have expiration dates and must be renewed

![[Pasted image 20250720122428.png]]

### Certificate Management
- **Certificate Type**: Load balancer uses X.509 certificates (SSL/TLS server certificates)
- **AWS Certificate Manager (ACM)**: Manage certificates using ACM
- **Custom Certificates**: Can create and upload your own certificates

### HTTPS Listener Configuration
- **Default Certificate**: Must specify a default certificate
- **Multiple Domains**: Add additional certificates to support multiple domains
- **SNI Support**: Clients can use Server Name Indication (SNI) to specify hostname

### Server Name Indication (SNI)
- **Problem Solved**: Loading multiple SSL certificates onto one web server
- **How it Works**: Client indicates target server hostname in initial SSL handshake
- **Certificate Selection**: Server finds correct certificate or returns default
- **Compatibility**: Only works for ALB, NLB, and CloudFront

![[Pasted image 20250720122804.png]]

## Connection Draining

### Feature Names
- **Connection Draining**: Classic Load Balancer (CLB)
- **Deregistration Delay**: Application Load Balancer (ALB) & Network Load Balancer (NLB)

### How It Works
- **Purpose**: Time to complete "in-flight requests" while instance is de-registering or unhealthy
- **Behavior**: Stops sending new requests to EC2 instance being de-registered
- **Duration**: Between 1 to 3600 seconds (default: 300 seconds)
- **Disable Option**: Can be disabled by setting value to 0
- **Best Practice**: Set low value if requests are short

![[Pasted image 20250720123211.png]]

## Auto Scaling Groups (ASG)

### Overview
- **Purpose**: Automatically adjust the number of EC2 instances based on demand
- **Real-world Need**: Website and application load changes over time
- **Cloud Advantage**: Can create and terminate servers quickly

### Goals of Auto Scaling Groups
- **Scale Out**: Add EC2 instances to match increased load
- **Scale In**: Remove EC2 instances to match decreased load
- **Instance Limits**: Ensure minimum and maximum number of instances
- **Load Balancer Integration**: Automatically register new instances to load balancer
- **Instance Replacement**: Re-create EC2 instances if previous ones are terminated (unhealthy)
- **Cost**: ASG service is free (only pay for EC2 instances)

![[Pasted image 20250720123456.png]]

![[Pasted image 20250720123528.png]]

### Launch Template
Configuration template that defines instance specifications for Auto Scaling Group.

![[Pasted image 20250720123624.png]]

### CloudWatch Integration
- **Scaling Triggers**: Scale ASG based on CloudWatch alarms
- **Metrics Monitoring**: Alarms monitor metrics (Average CPU, custom metrics)
- **Instance-Level Metrics**: Metrics computed for overall ASG instances
- **Scale-Out Policy**: Increase instances based on alarms
- **Scale-In Policy**: Decrease instances based on alarms

![[Pasted image 20250720123739.png]]

### Scaling Policies

#### Dynamic Scaling

**Target Tracking Scaling**:
- Simple to set up
- Example: Maintain average ASG CPU at around 40%

**Simple/Step Scaling**:
- Triggered by CloudWatch alarms
- Example: Add 2 units when CPU > 70%, remove 1 unit when CPU < 30%

#### Scheduled Scaling
- **Use Case**: Anticipate scaling based on known usage patterns
- **Example**: Increase minimum capacity to 10 at 5pm on Fridays

#### Predictive Scaling
- **Function**: Continuously forecast load and schedule scaling ahead
- **Benefit**: Proactive rather than reactive scaling

### Scaling Metrics
**Good Metrics for Scaling**:
- CPU Utilization
- RequestCountPerTarget
- Average Network In/Out

### Scaling Cooldown
- **Duration**: After scaling activity, cooldown period is 300 seconds (default)
- **Behavior**: ASG will not launch or terminate additional instances during cooldown
- **Purpose**: Allow metrics to stabilize
- **Best Practice**: Use ready-to-use AMI to reduce configuration time and serve requests faster, reducing cooldown period