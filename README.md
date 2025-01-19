# MONITORING-AND-ALERTING-FOR-EC2-LOGIN-FAILURE-ATTEMPTS-USING-AMAZON-CLOUDWATCH

# 1. Create an Amazon Linux 2 EC2 Instance
- Create an Amazon Linux 2 EC2 Instance with the necessary security groups allowing SSH and relevant ports
- Attach an IAM role to the instance that has the following permissions:
```
CloudWatchAgentServerPolicy
AmazonEC2ReadOnlyAccess
CloudWatchFullAccess
```
# 2. Configure the EC2 Instance for CloudWatch Logs
Update the Instance and Install the CloudWatch Agent:
- SSH into your EC2 instance:
```bash
ssh -i your-key.pem ec2-user@your-instance-ip
```
- Update the system:
```bash
sudo yum update -y
```
- Install the CloudWatch Agent:
```bash
sudo yum install amazon-cloudwatch-agent -y
```
Configure the CloudWatch Agent:
-	Create the agent configuration file:
```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
```
-	Add the following configuration to monitor /var/log/secure (log file for login events on Amazon Linux):
```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/secure",
            "log_group_name": "LoginFailures",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```
Notes: Specifies what log files to monitor ```(/var/log/secure)``` and where to send them in CloudWatch ```(log group: LoginFailures)```
-	Save and exit

Start the CloudWatch Agent:
-	Run the following command:
```
