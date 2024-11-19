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

### SCRIPT
The Ansible Playbook Structure:  

a. The playbook in playbook.yml defines roles for both Linux and Windows hosts.
b. The when conditions are used to apply roles based on the OS family.
c. Linux Tasks: The tasks in main.yml  install, configure, and start the CloudWatch Agent on Linux.
d. Windows Tasks: The tasks in main.yml  download, install, configure, and start the CloudWatch Agent on Windows.
e. Template Lookup: The lookup('template', 'cloudwatch-agent-config.json.j2') is used to fetch the configuration template.

### Architecture Diagram


### Logical Steps
1. Use AWS Organization, create centralized account for Automations. In this account:  
a. Create EC2 to host Ansible Playbook. Ensure the IAM role attached to the EC2 instance running the playbook has the necessary permissions:*SSM Parameter Store Access, STS AssumeRole, EC2 Describe Instances, SSM Send Command*  
b. Create SSM Parameter Store to store Account IDs.
c. Make use of CloudFormation Stack Sets and deploy IAM required for cross account access in all existing accounts. Configure the StackSet to automatically deploy the stack to new accounts added to the organization.    
d. Use ansible playbook to create roles for EC2 to send data to cloudwatch and attach these roles to ec2. *Alternatively, modify attached role to grant permisison to push logs to central cloudwatch.*

2. Ansible are idempotent in nature, thus we can schedule to run the playbook everyday. 





#### Automating addition of new AWS Accounts
1. Use AWS CloudFormation StackSets to automate the creation of IAM roles in newly added accounts, streamlining integration. CloudFormation StackSets simplify the configuration of cross-accounts permissions and allow for automatic creation and deletion of resources [when accounts are joining or are removed from your Organization](https://aws.amazon.com/blogs/aws/new-use-aws-cloudformation-stacksets-for-multiple-accounts-in-an-aws-organization/). Goal is to give one or more of SSH access, Ability to create Delete keys, SSM access, Describe EC2.




