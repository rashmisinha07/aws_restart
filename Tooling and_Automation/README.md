# Using AWS Systems Manager
AWS Systems Manager is a collection of capabilities that I can use to centralize operational data and automate tasks across your Amazon Web Services (AWS) resources. Systems Manager can configure and manage Amazon Elastic Compute Cloud (Amazon EC2) instances, on-premises servers, virtual machines, and other AWS resources at scale. 
# Generate inventory lists for managed instances
I can use Fleet Manager, a capability of Systems Manager, to collect operating system information, application information, and metadata from EC2 instances, on-premises servers, or virtual machines in a hybrid environment.I can also use Fleet Manager to query metadata to quickly understand which instances are running the software and configurations that your software policy requires and which instances I need update.

I used Fleet Manager to gather inventory from an EC2 instance.

* **Step-1** In the AWS Management Console, in the  search box, enter Systems Manager and press Enter. This option takes  to the Systems Manager console page.
* **Step-2** In the left navigation pane, for **Node Management**, choose **Fleet Manager**.
