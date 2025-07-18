- Identity and Access Management - Global service
- Root Accounts create users 
- Can create groups, groups can only contains users, not other groups
- a user can belong to multiple groups![[Pasted image 20250715195639.png]]
- Allow user and groups to assign JSON documents - policies
- ![[Pasted image 20250715195725.png]]
- Apply the least privilege principle: don't give more permissions than a user needs

### IAM Policies Inheritance
![[Pasted image 20250715200345.png]]
	Policies can be inherited and combined in ,multiple user groups
### IAM Policies Structure
![[Pasted image 20250715200504.png]]

### Iam Roles
- Some AWS services will need to perform actions on your behalf
- We need to assign permissions to aws services
- EC2 Instance Roles, Lambda functions roles

Quiz
https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03/learn/quiz/5334490#overview