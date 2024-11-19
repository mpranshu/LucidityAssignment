# Lucidity Assignment
Case study : Multi-Account AWS EC2 Disk Utilization Monitoring

Issue: A company which has now acquired companies now manages 3 accounts. The company faces challenges in monitoring and alerting for the Disk Utilization of EBS attached to these EC2 instances. The company plans to further expand and monitoring disk utilzation might become a nightmare.

Goal: Centralized, Digestable, Expandable and Comprehensive solution to monitor disk utilization across multiple accounts using Ansible for collecting logs.

Additionally: 
1. Enrollment of new EC2 and accounts should be easy.
2. Should be able to collect Disk Util Data for both Linux and Windows machine.
3. Alerting when Disk Utilization is high / Free Disk Space is low.
4. Automation to the extent possible.
5. Follow Best Practices for IAM.
6. Low manual intervention after one time set-up.
7. If some failure occurs, system should be able to recover eassily and automatically. 