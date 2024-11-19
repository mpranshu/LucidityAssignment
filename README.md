# Lucidity Assignment
## Case study: Multi-Account AWS EC2 Disk Utilization Monitoring

#### Issue: 
A company which has now acquired companies now manages 3 accounts. The company faces challenges in monitoring and alerting for the Disk Utilization of EBS attached to these EC2 instances. The company plans to further expand and monitoring disk utilzation might become a nightmare.

#### Goal: 
Centralized, Digestable, Expandable and Comprehensive solution to monitor disk utilization across multiple accounts using Ansible for collecting logs.

#### Additionally: 
1. Enrollment of new EC2 and accounts should be easy.
2. Should be able to collect Disk Util Data for both Linux and Windows machine.
3. Alerting when Disk Utilization is high / Free Disk Space is low.
4. Automation to the extent possible.
5. Follow Best Practices for IAM.
6. Low manual intervention.


## Ideating
### Disk Utilization  
Free disk space is available as a metric in Amazon CloudWatch for EC2 instances, **but it is not provided out of the box**. To monitor disk space usage, including free disk space, you need to use the **CloudWatch Agent** or a custom script to push the metric to CloudWatch.

From this perspective, there are two potential solutions:  
1. **Developing and Running a Custom Script:** This involves creating a script that runs continuously, connects to AWS, and pushes logs to CloudWatch.  
2. **Using the CloudWatch Agent:** This option minimizes development effort, as the agent is developed and maintained by AWS, providing assurance of security, performance, and reliability in a production environment.

### Ansible
In the assignment, its highlighted that *"Before investing
into other tools, the company has decided to use Ansible to perform the required metric collection."* Further, Ansible is an open-source IT automation tool that simplifies tasks like configuration management, application deployment, and infrastructure provisioning using YAML-based Playbooks.

The following are key areas for configuration and usage of Ansible:  
1.a. Write a Playbook to deploy a custom script for Disk Utilization Collection  
--or--  
1.b. Write a Playbook to deploy the CloudWatch Agent on EC2 instances to collect disk space metrics.  
2. Ansible Node by design will also need access to run commands, install agent or script on the remote EC2 Instance. We need to provision this access. There are two ways to do it:  
a. **SSH Access**: Store Keys, IP/HostName in the Ansible Inventory. Use vaults for secure storage of confedential material. Ensure, SSH port is accessible on remote machine from Ansible Node.  
b. **SSM Access**: For EC2 instances that are managed by AWS, you can use AWS Systems Manager (SSM) to execute commands on instances without needing SSH access. Amazon Linux 2, 2023, some windows machine.

Without diving even further, it becomes there are multiple architectural choices that exist. Atleast 2x2=4 to start with(Custom Script or CWAgent and SSH Access or SSM Agent)

**There are probably many other options, but for sake of this assignment I will choose just one combination: CW Agent with SSM and not brainstoming on other possible option.**

### Explaination
### The Ansible Playbook Structure:

The Ansible playbook in this project is designed to automate the deployment and configuration of the CloudWatch Agent on EC2 instances across multiple AWS accounts. The playbook leverages AWS Organizations, CloudFormation StackSets, and Ansible's idempotent nature to ensure consistent and repeatable deployments. Below is a brief description of the key components and files in the scripts directory:

#### 1. `playbook.yml`
This is the main Ansible playbook that orchestrates the entire deployment process. It includes the following key sections:
- **Read Account IDs**: Reads AWS account IDs from the SSM Parameter Store.
- **Assume Role and Run Tasks**: Assumes roles in each account and runs tasks.
- **Create IAM Roles and Instance Profiles**: Creates IAM roles and instance profiles required for EC2 instances to push logs to CloudWatch.
- **Attach IAM Roles to EC2 Instances**: Attaches the created IAM roles to the EC2 instances.
- **Install and Configure CloudWatch Agent**: Installs and configures the CloudWatch Agent on both Linux and Windows EC2 instances.

#### 2. `assume_role_and_run_tasks.yml`
This file contains tasks that are included in the main playbook to assume roles in each AWS account and run specific tasks. Key tasks include:
- **Assume Role**: Uses the `sts_assume_role` module to assume a role in the target account.
- **Set AWS Credentials**: Sets the assumed role credentials as facts.
- **Run Tasks Using SSM**: Executes commands on EC2 instances using AWS Systems Manager (SSM).

#### 3. `linux_cloudwatch_agent/tasks/main.yml`
This file defines the tasks to install and configure the CloudWatch Agent on Linux EC2 instances. Key tasks include:
- **Install CloudWatch Agent**: Installs the CloudWatch Agent package.
- **Create Configuration File**: Copies the CloudWatch Agent configuration file to the appropriate location.
- **Start CloudWatch Agent**: Starts the CloudWatch Agent service.

#### 4. `windows_cloudwatch_agent/tasks/main.yml`
This file defines the tasks to install and configure the CloudWatch Agent on Windows EC2 instances. Key tasks include:
- **Install CloudWatch Agent**: Installs the CloudWatch Agent MSI package.
- **Create Configuration File**: Copies the CloudWatch Agent configuration file to the appropriate location.
- **Start CloudWatch Agent**: Starts the CloudWatch Agent service using PowerShell.

#### 5. `templates/cloudwatch-agent-config.json.j2`
This is a Jinja2 template file for the CloudWatch Agent configuration. It defines the metrics and logs to be collected and specifies the centralized CloudWatch Logs account as the destination for the logs.

### Logical Steps

1. **Use AWS Organization**:
   - Create a centralized account for automations.
   - Create an EC2 instance to host the Ansible playbook. Ensure the IAM role attached to the EC2 instance has the necessary permissions: SSM Parameter Store Access, STS AssumeRole, EC2 Describe Instances, SSM Send Command.
   - Create Centralized Account for monitoring and logging.

2. **Create SSM Parameter Store**:
   - Store the account IDs in the SSM Parameter Store.

3. **Use CloudFormation StackSets**:
   - Deploy IAM roles required for cross-account access in all existing accounts.
   - Configure the StackSet to automatically deploy the stack to new accounts added to the organization.

4. **Run Ansible Playbook**:
   - Use the Ansible playbook in centralized automation account for fetching EC2 basis tags, create roles for EC2 instances to send data to CloudWatch and attach these roles to the instances.
   - The same Ansible playbook can install cloudwatch agent.

5. **Automating Addition of New AWS Accounts and EC2**:
    - Use CloudWatch Events to trigger a Lambda function whenever a new account is added to the organization or an EC2 is created or tags of EC2 is modified.
    - The Lambda function can assume the necessary roles and execute the Ansible playbook to configure the new account, ec2 instance
    - Add new account information in SSM Parameter Store using the lambda function whwenever new account is added.
    - These can be placed in AWS Step Functions for easy EDA.




### Architecture Diagram
