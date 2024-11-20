# Lucidity Assignment
## Case study: Multi-Account AWS EC2 Disk Utilization Monitoring

#### Problem Statement: 
The company, which has grown through acquisitions, now manages three separate AWS accounts, each containing numerous EC2 instances. The CTO is concerned about potential disk space issues across these instances and requires a comprehensive solution to monitor disk utilization. The company prefers to use Ansible, an internally used configuration management tool, to perform the required metric collection before investing in other tools.



#### Requirement: 
The goal is to design and implement a comprehensive solution to monitor disk utilization across EC2 instances in multiple AWS accounts using Ansible. This solution should centralize access and management, aggregate collected data into a single, easily digestible format, and be scalable to accommodate additional AWS accounts as the company grows.



### Designing a Disk Utilization Monitoring Solution with Ansible

#### Ansible Playbook Options:

Option 1a: Write a playbook to deploy a custom script for disk utilization collection.  
Option 1b: Write a playbook to deploy the CloudWatch Agent on EC2 instances to collect disk space metrics.

#### Provisioning Access for Ansible Node(assumed to be EC2):

The Ansible node needs access to run commands, install agents, or scripts on the remote EC2 instances. There are two ways to provision this access:  
- **SSH Access**: Store keys, IP/HostName in the Ansible inventory. Use vaults for secure storage of confidential material. Ensure the SSH port is accessible on the remote machine from the Ansible node.
- **SSM Access**: For EC2 instances managed by AWS, you can use AWS Systems Manager (SSM) to execute commands on instances without needing SSH access. This is supported on Amazon Linux 2, 2023, and some Windows machines. In other machines, SSM Agent can be installed as part of boarding(standard process)  

#### Architectural Choices:

There are multiple architectural choices to consider. At least 2x2=4 combinations to start with (Custom Script or CloudWatch Agent and SSH Access or SSM Agent).
For the sake of this assignment, I will choose one combination: CloudWatch Agent with SSM Access. Which seems a good option to start with though with obvious laggings.

### Detailed Solution Overview
#### The Ansible Playbook Structure:

The Ansible playbook in this project is designed to automate the deployment and configuration of the CloudWatch Agent on EC2 instances across multiple AWS accounts. The playbook leverages AWS Organizations, CloudFormation StackSets, and Ansible's idempotent nature to ensure consistent and repeatable deployments. Below is a brief description of the key components and files in the scripts directory:

 1. `playbook.yml`
This is the main Ansible playbook that orchestrates the entire deployment process. It includes the following key sections:
    - **Read Account IDs**: Reads AWS account IDs from the SSM Parameter Store.
    - **Assume Role and Run Tasks**: Assumes roles in each account and runs tasks.
    - **Create IAM Roles and Instance Profiles**: Creates IAM roles and instance profiles required for EC2 instances to push logs to CloudWatch.
    - **Attach IAM Roles to EC2 Instances**: Attaches the created IAM roles to the EC2 instances.
    - **Install and Configure CloudWatch Agent**: Installs and configures the CloudWatch Agent on both Linux and Windows EC2 instances.

2. `assume_role_and_run_tasks.yml`
This file contains tasks that are included in the main playbook to assume roles in each AWS account and run specific tasks. Key tasks include:
    - **Assume Role**: Uses the `sts_assume_role` module to assume a role in the target account.
    - **Set AWS Credentials**: Sets the assumed role credentials as facts.
    - **Run Tasks Using SSM**: Executes commands on EC2 instances using AWS Systems Manager (SSM).

 3. `linux_cloudwatch_agent/tasks/main.yml`
    This file defines the tasks to install and configure the CloudWatch Agent on Linux EC2 instances. Key tasks include:
    - **Install CloudWatch Agent**: Installs the CloudWatch Agent package.
    - **Create Configuration File**: Copies the CloudWatch Agent configuration file to the appropriate location.
    - **Start CloudWatch Agent**: Starts the CloudWatch Agent service.

4. `windows_cloudwatch_agent/tasks/main.yml`
This file defines the tasks to install and configure the CloudWatch Agent on Windows EC2 instances. Key tasks include:
    - **Install CloudWatch Agent**: Installs the CloudWatch Agent MSI package.
    - **Create Configuration File**: Copies the CloudWatch Agent configuration file to the appropriate location.
    - **Start CloudWatch Agent**: Starts the CloudWatch Agent service using PowerShell.

5. `templates/cloudwatch-agent-config.json.j2`
This is a template file for the CloudWatch Agent configuration. It defines the metrics and logs to be collected and specifies the centralized CloudWatch Logs account as the destination for the logs. 

#### Implementation Steps

1. **Use AWS Organization**:
   - Create a centralized account for automations.
   - Create or use existing EC2 instance to host the Ansible playbook. Ensure the IAM role attached to the EC2 instance has the necessary permissions: SSM Parameter Store Access, STS AssumeRole, EC2 Describe Instances, SSM Send Command.
   - Create Centralized Account for monitoring and logging.

2. **Create SSM Parameter Store**:
   - Store the account IDs in the SSM Parameter Store.

3. **Use CloudFormation StackSets**:
   - Deploys IAM roles required for cross-account access in all existing accounts.
   - Configure the StackSet to automatically deploy the stack to new accounts added to the organization.

4. **Run Ansible Playbook**:
   - Use the Ansible playbook in centralized automation account for fetching EC2 basis tags, create roles for EC2 instances to send data to CloudWatch and attach these roles to the instances.
   - The same Ansible playbook can install cloudwatch agent.

5. **Automating Addition of New AWS Accounts and EC2**:
    - Use CloudWatch Events to trigger a Lambda function whenever a new account is added to the organization or an EC2 is created or tags of EC2 is modified.
    - The Lambda function can assume the necessary roles and execute the Ansible playbook to configure the new account, ec2 instance
    - Add new account information in SSM Parameter Store using the lambda function whenever new account is added. Execute Lambda function basis this event.
    - These can be placed in AWS Step Functions for easy Event Driven Architecture.

6. **Create Alerts**
    - Create CloudWatch Metrics, Alarms basis threshold, SNS Topic basis design using CloudFormation or Ansible to set alerts in central account

### Additional Configuration Steps(*if required*)
1. **Send Logs to S3 as well**
2. **Set Quicksight if required if CloudWatch is not something that is preffered for visualization**

### Component Summary
- AWS Organizations: Centralized management of multiple AWS accounts.
- IAM Roles and Policies: Grant access to EC2 instances across accounts.
- CloudWatch Agent: Collects and sends disk utilization metrics to CloudWatch.
- CloudWatch Logs and Metrics: Stores and aggregates disk utilization data.
- AWS Lambda, EventBridge:
Trigger Ansible basis events
- SNS: Used for alerting
- Ansible: Automates the deployment and configuration of the CloudWatch Agent and the creation of metric filters and alarms.(Can also be done by Lambda)

### Areas Addressed
1. Centralized Access and Management:
- Use AWS Organizations to manage multiple AWS accounts centrally.
- Create a centralized account for automation and monitoring.
- Use AWS IAM roles and policies to grant the centralized account access to the EC2 instances in the other accounts.
- Centralized Logging, Monitoring and Automation accounts

2. Data Aggregation:
- Use CloudWatch Logs and CloudWatch Metrics to collect and store disk utilization data from EC2 instances.
- Configure the CloudWatch Agent on each EC2 instance to send disk utilization metrics to CloudWatch.
- Use CloudWatch Metrics and Alerts to aggregate and process the collected data.
- Data can be sent S3 for other processing

3. Scalability:
- Use AWS CloudFormation StackSets to deploy IAM roles and policies across all existing and new AWS accounts.
- Automate the addition of new accounts and EC2 instances using CloudWatch Events,  AWS Lambda and Ansible

[Additional options available](https://aws.amazon.com/blogs/mt/keeping-ansible-effortless-with-aws-systems-manager/)  

  [Can be done without Ansible for scalable solution](https://aws.amazon.com/blogs/compute/automating-amazon-ec2-windows-ebs-volumes-monitoring-and-creating-alarms/)
### Architecture Diagram
The below represent account planning structure:
![Account Structure](Architecture%20Diagram/Account%20Structure.png)

High-level architecture:
![High Level Architecture](Architecture%20Diagram/Architecture%20Diagram.png)