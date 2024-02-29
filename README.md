# Install and Configure the AWS CLI
I installed and configured the AWS CLI on a Red Hat Linux instance because this instance type does not have the AWS CLI pre-installed. Some instance types, such as Amazon Linux, do come pre-installed with the AWS CLI.
I established a Secure Shell (SSH) connection to the instance.
I configured the installation with an access key that can connect to an AWS account. Finally, I used the AWS CLI to interact with AWS Identity and Access Management (IAM).
This is the diagram:
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7b66c19a-5e14-4068-a441-d3ae54a5e94a)

In the preceding diagram, I can access the AWS Cloud through an SSH connection. Within the AWS Cloud, a virtual private cloud (VPC) with a Red Hat EC2 instance is configured with the AWS CLI. IAM is configured, and I used the AWS CLI to interact with IAM.
#Connect to the Red Hat EC2 instance by using SSH
 I log in to an existing EC2 instance
 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f4b677f8-76c3-427e-aca3-a602fdccd025)
 
I am using window.
Step1: Download the ppk and save the labsuser.ppk file.
Step 2: Copy and paste the PublicIP into a text editor
Step 3: Download PuTTY to use an SSH utility to connect to the EC2 instance
Step 4: Open putty.exe.
Step 5: Configure the PuTTY timeout to keep the PuTTY session open for a longer period of time:
                Choose Connection.
                For Seconds between keepalives, enter 30
Step 6: Configure the PuTTY session:
              Choose Session.
              For the Host Name (or IP address), enter the PublicIP address that I copied from the previous steps.
               In PuTTY in the Connection list, choose SSH 
               Choose Auth
                Choose Browse.
                Browse to and select the labuser.ppk file that I downloaded.
                To choose the file, choose Open.
Step 7: In the PuTTY Security Alert window, choose Accept to trust and connect to the host.
Step 8: When prompted with login as, enter ec2-user and press Enter.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7fc34eed-2a3b-4847-b500-85136b327173)
