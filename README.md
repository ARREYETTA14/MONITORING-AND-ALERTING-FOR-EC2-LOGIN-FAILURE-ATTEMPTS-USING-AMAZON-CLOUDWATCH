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
**Notes:** Specifies what log files to monitor ```(/var/log/secure)``` and where to send them in CloudWatch ```(log group: LoginFailures)```
-	Save and exit

Start the CloudWatch Agent:
-	Run the following command:
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
```
-	Verify that the agent is running:
```bash
sudo systemctl status amazon-cloudwatch-agent
```

# 3. Set Up CloudWatch Alarms for Login Failures
- Navigate to the CloudWatch Console
- From the left-hand menu, go to **Logs** > **Log group**


- Locate and select the LoginFailures log group (this was created when you configured the CloudWatch agent to send logs)


- Inside the LoginFailure log group, create a **metric filter**


- Define the **pattern** as follows:
```
[timestamp=*Z, id, event=LOGIN_FAILED, user]
```

**Explanation of the Pattern:**

timestamp=*Z: Captures the timestamp in log entries.
id: Placeholder for the unique identifier of the log entry.
event=LOGIN_FAILED: Matches log entries indicating failed login attempts.
user: Captures the username involved in the failed login.

# 4. Assign a Metric to the Filter

Specify a **Metric Namespace**:
•	Example: CustomNamespace.
Provide a **Metric Name**:
•	Example: LoginFailuresMetric.
Define a **Metric Value**:
•	Use the default value of 1 (each log event matching the filter increases the count by 1).


Click **Create Metric Filter**

# 5. Create an Alarm

- In the CloudWatch console, go to **Alarms** > **In Alarm** > **Create alarm**.
- Click **Select Metric**
- Choose **Logs** > **Login Group Metric** > **LoginFailures – IncomingLogEvents**

Configure the Alarm Conditions:
•  Set the **Statistic** to **Sum** (to aggregate the total login failures over the evaluation period).
•  Set the **Period** to **1 minute.**
•  Define the threshold:
  •	Threshold type: **Static**
  •	Specify the threshold: **Greater than or equal to 5**

Configure action:
•	Alarm state trigger – In alarm
•	Select Create New Topic
•	Pass the Topic Name and the email endpoint
•	Click Create Topic

Hit next till everything is created.

# 6. Testing 

**Simulate Login Failures:**
•	Try to SSH into the instance with incorrect credentials about 5 times and observe the alarm behaviour. Check your CloudWatch alarm menu and your email endpoint






