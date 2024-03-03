# Creating Amazon EC2 Instances
## Overview 

AWS provides multiple ways to launch Amazon Elastic Compute Cloud (Amazon EC2) instance. 

I used the AWS Management Console to launch an EC2 instance and then use it as a bastion host to launch another EC2 instance, which will be a web server. I use EC2 Instance Connect to securely connect to the bastion host and use the AWS Command Line Interface (AWS CLI) to launch a web server instance.

The following diagram illustrates the final architecture that you will build:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e5b5f615-1eb7-49e6-b9a0-ac491727d13f)

## Launching an EC2 Instance by using the AWS Management Console

I launch an EC2 instance by using the AWS Management Console. The instance will be a bastion host from which I can use the AWS CLI.

* **Step 1:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **Amazon EC2 Management Console**.
  
* **Step 2:** From the **Launch instance** dropdown list, choose **Launch instance** to open the **Launch an instance** menu.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/74aaf39e-64b0-48e5-9643-ae14ca3af89e)

## Choose name and tags

I use tags to categorize my AWS resources in different ways, such as by purpose, owner, or environment. This categorization is useful when I have many resources of the same type; I can quickly identify a specific resource by its tags. Each tag consists of a key and a value, both of which I define.

When my name my instance, AWS creates a key-value pair. The key for this pair is Name, and the value is the name that I enter for my EC2 instance.

* **Step 3:** In the Name and tags section, for Name, enter `Bastion host`

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ab0b5d46-f57d-4421-bac7-a0ea1fbe05fd)


  ## Choose an AMI

  In this step, I choose an Amazon Machine Image (AMI). An AMI includes the following:

 *  A template for the root volume for the instance (for example, an operating system or an application server with applications)

* Launch permissions that control which AWS accounts can use the AMI to launch instances

* A block device mapping that specifies the volumes to attach to the instance when it is launched

  The Quick Start list contains the most commonly used AMIs. I can also create your own AMI or select an AMI from the AWS Marketplace, an online store where I can sell or buy software that runs on AWS.

 bastion host will use Amazon Linux 2.

 * **Step 4:** In the **Application and OS Images (Amazon Machine Image)** section, for **Quick Start**, confirm that **Amazon Linux** is selected. Keep this selection.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a9557d07-731a-4b2c-a8b0-0f9c3ec54b83)


   **Note:** This option corresponds to Amazon Linux 2 AMI (HVM) as indicated in the **Description**.

   ## Choose an instance type

   In this step, I choose an instance type, which determines the resources that will be allocated to my EC2 instance. Each instance type allocates a combination of virtual CPU, memory, disk storage, and network performance.

Instance types are divided into families, such as compute optimized, memory optimized, and storage optimized. The name of the instance type includes a family identifier, such as T3 and M5. The number indicates the generation of the instance, so M5 is newer than M4.

My application uses a t3.micro instance type, which is a small instance that can burst above baseline performance when it is busy. It is suitable for development and testing purposes and for applications that have bursty workloads.


* **Step 5:** From the Instance type dropdown list, search for and choose t3.micro.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/acc1d9c5-1a5e-47f5-91ae-3272acc065ce)

  ## Configure a key pair
  Amazon EC2 uses public key cryptography to encrypt and decrypt login information. To log in to my instance, I must create a key pair, specify the name of the key pair when I launch the instance, and provide the private key when I connect to the instance.

I use EC2 Instance Connect to log in to my instance, so I do not require a key pair.

* **Step 6:** In the **Key pair (login)** section, from the Key pair name - required dropdown list, choose **Proceed without key pair (Not recommended)**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/af62a21d-2498-4b92-b69a-ad18b33c50f4)

## Configure the network settings

I use this pane to configure networking settings.

The virtual private cloud (VPC) indicates which VPC I want to launch the instance into. I can have multiple VPCs, including different ones for development, testing, and production.

I launch the instance in a public subnet within the Lab VPC network.

* **Step 7:** In the **Network settings** section, choose **Edit**.
* **Step 8:**  From the **VPC - required** dropdown list, choose **Lab VPC**.
  The Lab VPC was created using an AWS CloudFormation template during the setup process of my lab. This VPC includes one public subnet.

 * **Step 9:** In the Subnet dropdown list, notice that Public Subnet is selected by default. Keep this default setting.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c5c1b25d-4d62-4adb-98a8-8dc970ca94e4)

* **Step 9:** In the **Auto-assign public IP** dropdown list, notice that **Enable** is selected by default. Keep this default setting.


 * **Step 10:** In the **Firewall (security groups)** section, notice that **Create security group** is selected. Configure the following options:

     * For **Security group name - required** enter `Bastion security group`
     * For **Description - required** enter `Permit SSH connections`
       
A security group acts as a virtual firewall that controls the traffic for one or more instances. When I launch an instance, I associate one or more security groups with the instance. I added rules to each security group that allow traffic to or from its associated instances. I can modify the rules for a security group at any time; the new rules are automatically applied to all instances that are associated with the security group.

   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b7af1d5c-d8df-4e6d-bd55-2fde18c9655b)

   ## Add storage
   
I can use this step to add additional Amazon Elastic Block Store (Amazon EBS) disk volumes and configure their size and performance.
I launch the EC2 instance using a default 8 gibibyte (GiB) disk volume. This is my root volume (also known as a boot volume).

* **Step 10:** In the Configure storage pane, keep the default storage configuration.

  ## Configure advanced details

* **Step 11:** Expand the **Advanced details** pane.

* **Step 12:**  From **IAM instance profile** dropdown list, choose **Bastion-Role**.
  The Bastion-Role profile grants permission to applications running on the instance to make requests to the Amazon EC2 service. This association of Role is required for the second half of this lab, where I use the AWS CLI to communicate with the Amazon EC2 service.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2435dde1-8890-4ce2-a1a3-83269ef47a55)


 * **Step 12:** Leave the default settings for all the other values.

   ## Launch an EC2 instance

   Now that I have configured my EC2 instance settings, it is time to launch my instance.

  * **Step 13:** In the **Summary** section, review the instance configuration details displayed, and choose **Launch instance**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e4fe2b61-19c4-44db-95cf-41eb058ea0bc)

* **Step 14:** Choose View all instances.

  # Logging in to the bastion host

  I use EC2 Instance Connect to log in to the bastion host that I just created.

 * **Step 15:** On the EC2 Management Console, from the list of EC2 instances displayed, choose the :ballot_box_with_check: check box for the bastion host instance.
   
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/87b6ff91-a664-43d1-bc8b-13c29370ab65)

  * **Step 16:** Choose Connect.
 * **Step 17:** On the EC2 Instance Connect tab, choose Connect to connect to the bastion host.
   
    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/075a3372-5e06-4df6-a07c-26b444205204)

   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8028216e-5ded-4eba-996e-aedbc1bcd8c5)


   # Launching an EC2 instance using the AWS CLI
   
I launch an EC2 instance using the AWS CLI. With the AWS CLI, I can automate the provisioning and configuration of AWS resources. Launching EC2 instance by using a CLI command is similar to launching an instance using the console. When I use a CLI command, I need to supply all the parameters to the command to successfully run it and launch.

I can enter this information or retrieve it from the environment using CLI commands (for example, the AMIId, SubnetId, and SecurityGroupId commands in this scenario).

I need to configure the new EC2 instance as a web server. In addition to providing the parameters previously mentioned, I also need to use the user data script to install the Apache web server.

I run the scripts or commands provided in the following section in the EC2 Instance Connect session.

## Retrieve the AMI to use

One of the parameters required when launching an instance is the AMI, which populates the boot disk of the instance. AWS continually patches and updates AMIs, so it is recommended to always use the latest AMI when launching instances.

I used AWS Systems Manager Parameter Store to retrieve the ID of the most recent Amazon Linux 2 AMI. AWS maintains a list of standard AMIs in Parameter Store, which makes it possible to automate this task.

* **Step 18:** Run the following script in your EC2 Instance Connect session:

        #Set the Region
         AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
         export AWS_DEFAULT_REGION=${AZ::-1}
         #Retrieve latest Linux AMI
         AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 
        'Parameters[0].[Value]' --output text)
         echo $AMI

 **Script explanation**

 * The script retrieves the Availability Zone for the running instance using instance metadata.

 * The script retrieves the Region from the Availability Zone and exports it into the environment for subsequent use.

 * The script calls the AWS Systems Manager (indicated with the ssm command) and uses the get-parameters command to retrieve 
  the AMI ID from Parameter Store.

 * The AMI requested was for Amazon Linux 2 (which is the same as the one used for bastion host).

 * The AMI ID has been stored in an environment variable called AMI.

 * If your EC2 Instance Connect session disconnects, it will lose the information stored in the environment variables. Refresh my browser to reconnect. If I do so, I will need to re-run all of the steps in this task, starting with the commands in this step, to obtain the AMI ID.


  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c6b1ba63-8da6-4908-856f-81dcf1b2cd68)


  ## Retrieve the subnet to use

I launch the new instance in the public subnet. When launching an instance, I can specify the subnet ID.

* **Step 19:** To retrieve the subnet ID for the public subnet, run the following command:


           SUBNET=$(aws ec2 describe-subnets --filters 'Name=tag:Name,Values=Public Subnet' --query Subnets[].SubnetId -- 
            output text)
            echo $SUBNET

  

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9eb57482-ad93-4d72-8df9-39964c6a0bf4)

  This script runs the aws ec2 command with the describe-subnets subcommand to retrieve the subnet ID of the subnet named Public Subnet.

  ## Retrieve the security group to use

  This lab includes a web security group, which allows inbound HTTP requests.

* **Step 20:**   Run the following command:


        SG=$(aws ec2 describe-security-groups --filters Name=group-name,Values=WebSecurityGroup --query 
        SecurityGroups[].GroupId --output text)
        echo $SG

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3ca3ddc4-7d75-49b0-bf46-55f56ef082da)


  The script runs the aws ec2 command with the describe-security-groups subcommand to retrieve the security group ID of the web security group.


  ## Download a user data script

  I launch an instance that acts as a web server. To install and configure the web server, I provide a user data script that automatically runs when the instance launches.

  * **Step 21:** To download the user data script, run the following command:

         wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1-23732/171-lab-JAWS-create- 
         ec2/s3/UserData.txt



    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/167bcf2b-716c-4287-8f20-9ede6bd48e4b)

  * **Step 22:**  To view the contents of the script, run the following command:

               cat UserData.txt


    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a59d6e1e-81ee-47e5-bac6-8448a10f8ec4)

    The script does the following:

 * Installs a web server
 * Downloads a .zip file containing the web application
 * Installs the web application

   ## Launch the instance

   I have all the necessary information required to launch the web server instance.

   * **Step 23:** Run the following command:

           INSTANCE=$(\
           aws ec2 run-instances \
           --image-id $AMI \
           --subnet-id $SUBNET \
           --security-group-ids $SG \
           --user-data file:///home/ec2-user/UserData.txt \
           --instance-type t3.micro \
           --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web Server}]' \
           --query 'Instances[*].InstanceId' \
           --output text \
           )
           echo $INSTANCE

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3ba7e115-38e5-4ab5-9055-b8b737cb7e41)


 The run-instances command launches a new instance using these parameters:

  * **Image:** Uses the AMI value obtained earlier from Parameter Store

* **Subnet:** Specifies the public subnet retrieved earlier and, by association, the VPC in which to launch the instance

* **Security group:** Uses the web security group retrieved earlier, which permits HTTP access

* **User data:** References the user data script that you downloaded, which installs the web application

* **Instance type:** Specifies the type of instance to launch

* **Tags:** Assigns a name tag with the value of **Web Server**

The query parameter specifies that the command should return the instance ID once the instance is launched.

The output parameter specifies that the output of the command should be in text. Other output options are json and table.

**Note:** The ID of the new instance has been stored in the INSTANCE environment variable.

## Wait for the instance to be ready

I can monitor the status of the instance by using the AWS Management Console or by querying the status by using the AWS CLI.

* **Step 24 :** Run the following command:

          aws ec2 describe-instances --instance-ids $INSTANCE

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0326fe12-a5f3-4dfa-8067-f60291daf1ec)

  All information related to the instance is displayed in JSON format. This information includes the instance status.

  I can retrieve specific information by using the query parameter.

* **Step 25 :** Run the following command:
 
             aws ec2 describe-instances --instance-ids $INSTANCE --query 'Reservations[].Instances[].State.Name' --output 
             text

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0c00a7e9-06d5-4614-aacc-b6d948c8afd8)


This command is the same as the command in the previous step, but rather than displaying all information about the instance, this command displays only the name of the instance state.

This command displays a status of pending or running.
Run this command again until it returns a status of running.


  ## Test the web server

  I can now test that the web server is working. I can retrieve a URL to the instance through the AWS CLI.

 * **Step 26 :** Run the following command:

         aws ec2 describe-instances --instance-ids $INSTANCE --query Reservations[].Instances[].PublicDnsName --output text

   
 

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0792db28-8bf6-4b0b-801c-bb4fab63d857)

This command returns the public IPv4 Domain Name System (DNS) name of the instance.

Copy the DNS name that is displayed. It should be similar to the following: 

ec2-34-219-32-112.us-west-2.compute.amazonaws.com

Paste the DNS name into a new web browser tab, and then press Enter.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/af6a533f-845c-43cc-8528-3b9caa5370ec)

A web page should be displayed, which demonstrates that the web server was successfully launched and configured.
I can also see the instance on the Amazon EC2 management console.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/05fcfaa4-d8db-443a-9ee1-b71c79c87d65)

# challenge 1: Connect to an EC2 instance

In this challenge, I troubleshoot the security configuration of an instance called Misconfigured Web Server.

The following are my tasks:

Try to connect to the Misconfigured Web Server instance by using EC2 Instance Connect.

Diagnose why this does not work, and fix the misconfiguration. 


The problem with challenge 1 is that the old security groups doesn’t work. I have tried connecting to it using ssh many times with no success. Then I changed the security groups with the Bastion security groups and I was able to connect to it.



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/806a822c-e984-4c7a-bbd6-c082d17f56f1)




![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/42455373-e076-4eb4-9c21-fe0f0cd8a683)




# Conclusion

I have successfully done the following:

 * Launched an EC2 instance by using AWS Management Console
 * Connected to the instance by using EC2 Instance Connect
 * Launched an EC2 instance by using the AWS CLI

 












    





  

 




