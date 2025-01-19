# MONITORING-AND-ALERTING-FOR-EC2-LOGIN-FAILURE-ATTEMPTS-USING-AMAZON-CLOUDWATCH

1. Create an Amazon Linux 2 EC2 Instance
- Create an Amazon Linux 2 EC2 Instance with the necessary security groups allowing SSH and relevant ports
- Attach an IAM role to the instance that has the following permissions:
```
CloudWatchAgentServerPolicy
AmazonEC2ReadOnlyAccess
CloudWatchFullAccess
```
