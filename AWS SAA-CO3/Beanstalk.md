### Typical architecture: web app 3-tier
- ![[Pasted image 20250726114324.png]]

### Developer problems on AWS
- Managing infrastructure
- Deploying Code
- Configuring all the databases, load balancers, etc
- Scaling concerns
- Most web apps have the same architecture (ALB + ASG)
- All developers want is for their code to run
- Possibly consistently across different applications and environments

### Beanstalk - overview
- Is a developer centric view of deploying an application on AWS
- uses all of the components we've seen before : ec2, asg, elb , rds,...
- Managed service
	- Automatically handles capacity provisioning, load balancing, scaling, application health monitoring, instance configuration,...
	- Just the application code is the responsibility of the developer
- We still have full control over the configuration
- beanstalk is free but you pay of the underlying instances

#### Components
- Application: collection of Beanstalk components (environments,versions, configurations,...)
- Application Version: an interation of your application code
- Environment:
	- Collection of AWS resources running an application version (only one application version at a time)
	- Tiers: web server environment tier and worker environment tier
	- You can create multiple environments (dev, test, prod)
	- ![[Pasted image 20250726114747.png]]
	- ![[Pasted image 20250726114756.png]]
	- ![[Pasted image 20250726114837.png]]
	- ![[Pasted image 20250726114855.png]]
- 