# Install and Configure the AWS CLI
I installed and configured the AWS CLI on a Red Hat Linux instance because this instance type does not have the AWS CLI pre-installed. Some instance types, such as Amazon Linux, do come pre-installed with the AWS CLI.
I established a Secure Shell (SSH) connection to the instance.
I configured the installation with an access key that can connect to an AWS account. Finally, I used the AWS CLI to interact with AWS Identity and Access Management (IAM).
This is the diagram:
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7b66c19a-5e14-4068-a441-d3ae54a5e94a)

In the preceding diagram, I can access the AWS Cloud through an SSH connection. Within the AWS Cloud, a virtual private cloud (VPC) with a Red Hat EC2 instance is configured with the AWS CLI. IAM is configured, and I used the AWS CLI to interact with IAM.
#Connect to the Red Hat EC2 instance by using SSH
 I log in to an existing EC2 instance.

 
 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f4b677f8-76c3-427e-aca3-a602fdccd025)

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/80c5c88d-6846-4f19-8e33-f7f3b7ebb03b)

 
I am using window.
* **Step 1:** Download the ppk and save the **labsuser.ppk** file.
* **Step 2:** Copy and paste the PublicIP into a text editor
* **Step 3:** Download PuTTY to use an SSH utility to connect to the EC2 instance
* **Step 4:** Open putty.exe.
* **Step 5:** Configure the PuTTY timeout to keep the PuTTY session open for a longer period of time:
                Choose Connection.
                For Seconds between keepalives, enter `30`
* **Step 6:** Configure the PuTTY session:
              Choose Session.
              For the Host Name (or IP address), enter the PublicIP address that I copied from the previous steps.
               In PuTTY in the Connection list, choose SSH 
               Choose Auth
                Choose Browse.
                Browse to and select the labuser.ppk file that I downloaded.
                To choose the file, choose Open.
* **Step 7:** In the PuTTY Security Alert window, choose Accept to trust and connect to the host.
* **Step 8:** When prompted with login as, enter `ec2-user` and press Enter.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7fc34eed-2a3b-4847-b500-85136b327173)

Now I connected to EC2 instance.

# Install the AWS CLI on a Red Hat Linux instance
I followed these steps from the terminal window to install the AWS CLI on a Red Hat Linux instance.

* **Step 9:** To write the downloaded file to the current directory, run the following curl command with the -o option:
  
                         curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  The following is an example of the output:
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d092ecc6-c82b-4e37-a4ac-2abf2f2f6b2d)

* **Step 10:** To unzip the installer, run the following unzip command with the -u option.

               unzip -u awscliv2.zip
  The following is an example of the output:
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cd040951-0df0-4505-9ce9-52d46df1c1ad)

* **Step 11:** To run the install program, run the following command. This sudo command grants write permissions to the directory.

               sudo ./aws/install
   The following is an example of the output:
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ae3f23c6-a4a4-4913-a4d3-d4bd4ffdb305)

 * **Step 12:** To confirm the installation, run the following command:

          aws --version
   The following is an example of the output:
   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f8c5ecfc-36d6-4c62-8b20-5131e92598ed)

* **Step 12:** To verify that the AWS CLI is now working, run the following aws help command. The help command displays the information for the AWS CLI.

         aws help
  
The following is an example of the output:
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6a5c923e-a35e-4b4a-a892-10604556ca00)

At the : prompt, enter q to exit.
# Observe IAM configuration details in the AWS Management Console

In the AWS Management Console, in the Search box, enter IAM and choose **IAM**. This option takes me to the IAM console page. In the navigation pane, choose **Users**, and then choose **awsstudent**. 



   

  

  
  

  
