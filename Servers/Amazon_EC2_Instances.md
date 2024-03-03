# Creating Amazon EC2 Instances
## Overview 

AWS provides multiple ways to launch Amazon Elastic Compute Cloud (Amazon EC2) instance. 

I used the AWS Management Console to launch an EC2 instance and then use it as a bastion host to launch another EC2 instance, which will be a web server. I use EC2 Instance Connect to securely connect to the bastion host and use the AWS Command Line Interface (AWS CLI) to launch a web server instance.

The following diagram illustrates the final architecture that you will build:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e5b5f615-1eb7-49e6-b9a0-ac491727d13f)

# Launching an EC2 Instance by using the AWS Management Console

I launch an EC2 instance by using the AWS Management Console. The instance will be a bastion host from which I can use the AWS CLI.

* **Step 1:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **Amazon EC2 Management Console**.
  
* **Step 2:** From the **Launch instance** dropdown list, choose **Launch instance** to open the **Launch an instance** menu.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/74aaf39e-64b0-48e5-9643-ae14ca3af89e)

# Choose name and tags

I use tags to categorize my AWS resources in different ways, such as by purpose, owner, or environment. This categorization is useful when I have many resources of the same type; I can quickly identify a specific resource by its tags. Each tag consists of a key and a value, both of which I define.

When my name my instance, AWS creates a key-value pair. The key for this pair is Name, and the value is the name that I enter for my EC2 instance.

* **Step 3:** In the Name and tags section, for Name, enter `Bastion host`

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ab0b5d46-f57d-4421-bac7-a0ea1fbe05fd)


  # Choose an AMI

  In this step, I choose an Amazon Machine Image (AMI). An AMI includes the following:

 *  A template for the root volume for the instance (for example, an operating system or an application server with applications)

* Launch permissions that control which AWS accounts can use the AMI to launch instances

* A block device mapping that specifies the volumes to attach to the instance when it is launched

  The Quick Start list contains the most commonly used AMIs. I can also create your own AMI or select an AMI from the AWS Marketplace, an online store where I can sell or buy software that runs on AWS.

 bastion host will use Amazon Linux 2.

 * **Step 4:** In the **Application and OS Images (Amazon Machine Image)** section, for **Quick Start**, confirm that **Amazon Linux** is selected. Keep this selection.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a9557d07-731a-4b2c-a8b0-0f9c3ec54b83)


   **Note:** This option corresponds to Amazon Linux 2 AMI (HVM) as indicated in the **Description**.

   # Choose an instance type

   In this step, I choose an instance type, which determines the resources that will be allocated to my EC2 instance. Each instance type allocates a combination of virtual CPU, memory, disk storage, and network performance.

Instance types are divided into families, such as compute optimized, memory optimized, and storage optimized. The name of the instance type includes a family identifier, such as T3 and M5. The number indicates the generation of the instance, so M5 is newer than M4.

My application uses a t3.micro instance type, which is a small instance that can burst above baseline performance when it is busy. It is suitable for development and testing purposes and for applications that have bursty workloads.


* **Step 5:** From the Instance type dropdown list, search for and choose t3.micro.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/acc1d9c5-1a5e-47f5-91ae-3272acc065ce)

  # Configure a key pair
  Amazon EC2 uses public key cryptography to encrypt and decrypt login information. To log in to my instance, I must create a key pair, specify the name of the key pair when I launch the instance, and provide the private key when I connect to the instance.

I use EC2 Instance Connect to log in to my instance, so I do not require a key pair.

* **Step 6:** In the **Key pair (login)** section, from the Key pair name - required dropdown list, choose **Proceed without key pair (Not recommended)**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/af62a21d-2498-4b92-b69a-ad18b33c50f4)

# Configure the network settings

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

   # Add storage
   
I can use this step to add additional Amazon Elastic Block Store (Amazon EBS) disk volumes and configure their size and performance.
I launch the EC2 instance using a default 8 gibibyte (GiB) disk volume. This is my root volume (also known as a boot volume).

* **Step 10:** In the Configure storage pane, keep the default storage configuration.

  # Configure advanced details

* **Step 11:** Expand the **Advanced details** pane.

* **Step 12:**  From **IAM instance profile** dropdown list, choose **Bastion-Role**.
  The Bastion-Role profile grants permission to applications running on the instance to make requests to the Amazon EC2 service. This association of Role is required for the second half of this lab, where I use the AWS CLI to communicate with the Amazon EC2 service.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2435dde1-8890-4ce2-a1a3-83269ef47a55)


 * **Step 12:** Leave the default settings for all the other values.

   # Launch an EC2 instance

   Now that I have configured my EC2 instance settings, it is time to launch my instance.

  * **Step 13:** In the Summary section, review the instance configuration details displayed, and choose Launch instance.




  

 




