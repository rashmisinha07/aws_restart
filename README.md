# aws_restart
# Install and Configure the AWS CLI
I installed and configured the AWS CLI on a Red Hat Linux instance because this instance type does not have the AWS CLI pre-installed. Some instance types, such as Amazon Linux, do come pre-installed with the AWS CLI.
I established a Secure Shell (SSH) connection to the instance.
I configured the installation with an access key that can connect to an AWS account. Finally, I used the AWS CLI to interact with AWS Identity and Access Management (IAM).
This is the diagram:
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7b66c19a-5e14-4068-a441-d3ae54a5e94a)

In the preceding diagram, I can access the AWS Cloud through an SSH connection. Within the AWS Cloud, a virtual private cloud (VPC) with a Red Hat EC2 instance is configured with the AWS CLI. IAM is configured, and I used the AWS CLI to interact with IAM.

