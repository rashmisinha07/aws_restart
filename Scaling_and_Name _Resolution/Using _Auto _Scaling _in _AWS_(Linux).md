# Using Auto Scaling in AWS (Linux)
## Overview

 I used the AWS Command Line Interface (AWS CLI) to create an Amazon Elastic Compute Cloud (EC2) instance to host a web server and create an Amazon Machine Image (AMI) from that instance. I used that AMI as the basis for launching a system that scales automatically under a variable load by using Amazon EC2 Auto Scaling. I also created an Elastic Load Balancer to distribute the load across EC2 instances created in multiple Availability Zones by the auto scaling configuration. 

 Starting architecture:

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1d0f6d77-c75d-43b3-95b1-792674fe6e42)

 Final architecture:

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e216ea1d-ddb1-4e99-b85b-f40a3cf0517e)


 # Creating a new AMI for Amazon EC2 Auto Scaling

 I launched a new EC2 instance and then create a new AMI based on that running instance. I used the AWS CLI on the Command Host EC2 instance to perform all of these operations.


## Connecting to the Command Host instance

I used EC2 Instance Connect to connect to the Command Host EC2 instance that was created when the lab was provisioned. I used this instance to run AWS CLI commands.

* **Step 1:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2`  to open the **EC2 Management Console**.

* **Step 2:** In the navigation pane, choose **Instances**.

* **Step 3:** From the list of instances, select the  **Command Host** instance.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c5786a12-8609-4f3c-a7ee-7b509af072a1)


* **Step 4:** Choose **Connect**.

* **Step 5:** On the **EC2 Instance Connect** tab, choose **Connect**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/de2b2cc1-9229-4b03-92e3-667b8fb3e8a3)


## Configuring the AWS CLI


The AWS CLI is preconfigured on the Command Host instance.

* **Step 6:** To confirm that the Region in which the Command Host instance is running is the same as the lab (the us-west-2 Region), run the following command:

      curl http://169.254.169.254/latest/dynamic/instance-identity/document | grep region



  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ab278176-bafb-4b75-85f4-a9ceb6d1f2ec)


   I used this Region information in the next steps.


* **Step 7:** To update the AWS CLI software with the correct credentials, run the following command:

               aws configure


* **Step 8:** At the prompts, enter the following information:

  * **AWS Access Key ID**: Press Enter.

  * **AWS Secret Access Key**: Press Enter.

  * **Default region name**: Enter the name of the Region from the previous steps in this
   task  (for example, `us-west-2`). If the Region is already displayed, press Enter.

  * **Default output format**: Enter `json`

Now I am ready to access and run the scripts detailed in the following steps.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3bd41c84-6354-42a6-be34-74bb0ba4ec29)


* **Step 9:** To access these scripts, enter the following command to navigate to their directory:

      cd /home/ec2-user/


## Creating a new EC2 Instance
I used the AWS CLI to create a new instance that hosts a web server. 

* **Step 10:** To inspect the UserData.txt script that was installed for me as part of the Command Host creation, run the following command:

        more UserData.txt

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/069d99aa-b31b-4d47-858a-f411587808cb)

This script performs a number of initialization tasks, including updating all installed software on the box and installing a small PHP web application that I can use to simulate a high CPU load on the instance. The following lines appear near the end of the script:




* **Step 11:** At the top of this page, choose **Details**, and choose **Show**.

* **Step 12:** Copy the **KEYNAME**,**AMIID**, **HTTPACCESS**, and **SUBNETID** values into a text editor document, and then choose **X** to close the **Credentials** panel.

* **Step 13:** In the following script, replace the corresponding text with the values from the previous step.

      aws ec2 run-instances --key-name KEYNAME --instance-type t3.micro --image-id AMIID --user-data file:///home/ec2-user/UserData.txt --security-group-ids HTTPACCESS --subnet-id SUBNETID --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]' --output text --query 'Instances[*].InstanceId'

  
 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6c18795d-7930-4261-8db3-5527c9d0a94f)

* **Step 14:** Enter your modified script into the terminal window, and run the script.

The output of this command provides me with an **InstanceId**. Subsequent steps in this lab refer to this value as **NEW-INSTANCE-ID**. Replace this value as needed throughout this lab. 

* **Step 15:** Copy and paste the **InstanceId** value into a text editor to use later. 

* **Step 16:** To use the aws ec2 wait instance-running command to monitor this instance's status, replace NEW-INSTANCE-ID in the following command with the InstanceID value that I copied in the previous step. Run  modified command. 
 

     aws ec2 wait instance-running --instance-ids i-04c4f4b65068aa24d

Wait for the command to return to a prompt before proceeding.

My instance starts a new web server. To test that the web server was installed properly, I must obtain the public DNS name.

* **Step 17:** To obtain the public DNS name, in the following command, replace NEW-INSTANCE-ID with the value that I copied previously, and run modified command:


        aws ec2 describe-instances --instance-id i-04c4f4b65068aa24d --query 
       'Reservations[0].Instances[0].NetworkInterfaces[0].Association.PublicDnsName'

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/20b8254c-339d-42b5-ab7d-6fdd40b65f4b)



* **Step 18:** Copy the output of this command without the quotation marks. 

The value of this output is referred to as **PUBLIC-DNS-ADDRESS** in the next steps. 

* **Step 19:** In a new browser tab, enter the output that I copied from the previous step.

It could take a few minutes for the web server to be installed. Wait 5 minutes before continuing to the next steps.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/11107b0a-2819-4d35-bc20-c2f5ef89646d)

Do not choose **Start Stress** at this stage. 

* **Step 20:** In the following command, replace PUBLIC-DNS-ADDRESS with the value that I copied in the previous steps, and then run  modified command.

      http://ec2-35-167-43-98.us-west-2.compute.amazonaws.com/index.php

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6c04d031-1817-40cf-9762-43a9d4f3cd8e)


  ## Creating a Custom AMI

  I created a new AMI based on that instance that I just created.

* **Step 21:** To create a new AMI based on this instance, in the following aws ec2 create-image command, replace NEW-INSTANCE-ID with the value that I copied previously, and run adjusted command:


     aws ec2 create-image --name WebServerAMI --instance-id i-04c4f4b65068aa24d


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/82b28c24-da7b-4345-9e4c-cb2fe98b6e97)



     By default, the aws ec2 create-image command restarts the current instance before creating the AMI to ensure the integrity of the image on the file system. While my AMI is being created, proceed to the next task.


# Creating an auto scaling environment

I created a load balancer that pools a group of EC2 instances under a single Domain Name System (DNS) address. I used auto scaling to create a dynamically scalable pool of EC2 instances based on the image that I created in the previous task. Finally, I created a set of alarms that scale out or scale in the number of instances in my load balancer group whenever the CPU performance of any machine within the group exceeds or falls below a set of specified thresholds.

I can perform the following task by using either the AWS CLI or the AWS Management Console. For this lab, I used the AWS Management Console.


## Creating an Application Load Balancer


I created a load balancer that can balance traffic across multiple EC2 instances and Availability Zones.

* **Step 22:** On the EC2 Management Console, in the left navigation pane, locate the Load Balancing section, and choose **Load Balancers**.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a62318cd-4ee7-49bd-b7ad-2921edfe6e3e)


* **Step 23:** Choose Create load balancer.

* **Step 24:** In the **Load balancer types** section, for **Application Load Balancer**, choose **Create**.

* **Step 25:** On the **Create Application Load Balancer** page, in the **Basic configuration** section, configure the following option:

   * For **Load balancer name**, enter `WebServerELB`


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5e02e81b-6164-4c85-9453-7ca2734de4bb)

* **Step 26:** In the **Network mapping** section, configure the following options:

   * For **VPC**, choose **Lab VPC**.

   * For **Mappings**, choose both Availability Zones listed.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2e047819-fce6-4142-89e9-385896934ccf)


   * For the first Availability Zone, choose **Public Subnet 1**.
   * For the second Availability Zone, choose **Public Subnet 2**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fb904e9c-35c4-483b-80d8-d969983ae6e8)


These options configure the load balancer to operate across multiple Availability Zones.


* **Step 27:** In the **Security groups** section, choose the **X** for the **default** security group to remove it.

From the **Security groups** dropdown list, choose **HTTPAccess**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8cd40f49-0f7d-4374-92eb-ac15629e0377)


The **HTTPAccess** security group has already been created for you, which permits HTTP access.

* **Step 28:** In the **Listeners and routing** section, choose the **Create target group** link.

**Note**: This link opens a new browser tab with the Create target group configuration options.

* **Step 29:** On the **Specify group details** page, in the **Basic configuration** section, configure the following options:

    * For **Choose a target type**, choose **Instances**.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b67b22ff-0465-449b-ab2a-22405c4c8c44)

     * For **Target group name**, enter `webserver-app`
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/96002c59-649f-455d-8232-490b1d83f862)

* **Step 30:** In the **Health checks** section, for **Health check path**, enter `/index.php`
  

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1ee1d1c4-d3de-4350-8039-5eece4c7a65c)

* **Step 31:** At the bottom of the page, choose **Next**.

* **Step 32:** On the **Register targets** page, choose **Create target group**.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9f38c476-c7ed-4794-a9e6-1a18079cdf48)


Once the target group has been created successfully, close the **Target groups** browser tab.

* **Step 33:** Return to the **Load balancers** browser tab, and locate the **Listeners and routing** section. For **Default action**, choose  **Refresh** to the right of the **Forward to** dropdown list.

* **Step 34:** From the **Forward to** dropdown list, choose **webserver-app**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c4313d3d-1373-4b19-8024-d640b1d9476c)


* **Step 35:** At the bottom of the page, choose **Create load balancer**.

   I should receive a message similar to the following:

   :heavy_check_mark: Successfully created load balancer: WebServerELB

* **Step 36:** To view the **WebServerELB** load balancer that you created, choose **View load balancer**.

* **Step 37:** To copy the DNS name of the load balancer, use the copy option , and paste the DNS name into a text editor. 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e08da093-57c8-4627-b7e2-8e9a34e23fac)

   I need this information later in the lab.


   # Creating a launch template
  
I  created a launch template for my Auto Scaling group. A launch template is a template that an Auto Scaling group uses to launch EC2 instances. When I create a launch template, I specify information for the instances, such as the AMI, instance type, key pair, security group, and disks.

* **Step 38:** On the EC2 Management Console, in the left navigation pane, locate the **Instances** section, and choose **Launch Templates**.

* **Step 39:** Choose **Create launch template**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b0882f8a-2617-41c8-bb7c-201c65c8c758)


* **Step 40:** On the Create launch template page, in the Launch template name and description section, configure the following options:

   * For **Launch template name - required**, enter `web-app-launch-template`

    * For **Template version description**, enter `A web server for the load test app`

    * For **Auto Scaling guidance**, :ballot_box_with_check: select  **Provide guidance to help me set up a template that I can use with EC2 Auto Scaling**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b95df79b-540d-434f-bcb5-e8e203e11bd0)


* **Step 41:** In the **Application and OS Images (Amazon Machine Image) - required** section, choose the **My AMIs** tab. 

Notice that **WebServerAMI** is already chosen.

* **Step 42:** In the **Instance type** section, choose the **Instance type** dropdown list, and choose **t3.micro**.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e0d69c2a-c674-435b-a9a2-358417b24660)

* **Step 43:** In the **Key pair (login)** section, confirm that the **Key pair name** dropdown list is set to **Don't include in launch template**.

 Amazon EC2 uses public key cryptography to encrypt and decrypt login information. To log in to my instance, I must create a key pair, specify the name of the key pair when I launch the instance, and provide the private key when I connect to the instance.

**Note:** In this lab, I do not need to connect to the instance.

* **Step 44:** In the **Network settings** section, choose the **Security groups** dropdown list, and choose **HTTPAccess**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/dc2f4aa4-12e1-4e59-b1f3-6639cb5c7ce0)


When I launch an instance, I can pass user data to the instance. The data can be used to run configuration tasks and scripts.

* **Step 45:** Choose **Create launch template**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/adfb62ca-4aeb-4513-98b4-353a4c351982)


I should receive a message similar to the following:

:heavy_check_mark: Successfully created web-app-launch-template.

* **Step 46:** Choose **View launch templates**.

# Creating an Auto Scaling group

I use my launch template to create an Auto Scaling group.

* **Step 47:** Choose  **web-app-launch-template**, and then from the **Actions ** dropdown list, choose **Create Auto Scaling group**. 

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d2eda40e-dc23-4e5f-af91-abcf3992e276)


* **Step 48:** On the **Choose launch template or configuration** page, in the Name section, for **Auto Scaling group name**, enter `Web App Auto Scaling Group`
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/230528a0-3900-4b1a-bd7f-4fb30b9d704c)

* **Step 49:** Choose **Next.**


* **Step 50:** On the **Choose instance launch options** page, in the **Network** section, configure the following options:

    * From the **VPC** dropdown list, choose **Lab VPC**.

     * From the **Availability Zones and subnets** dropdown list, choose **Private Subnet 1 (10.0.2.0/24)** and **Private Subnet 2 (10.0.4.0/24)**. 


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b23169e5-8a00-4492-bbef-138b96815b8e)

* **Step 51:** Choose **Next**.


* **Step 52:** On the **Configure advanced options – optional** page, configure the following options: 

    * In the **Load balancing – optional** section, choose **Attach to an existing load balancer**.
     * In the **Attach to an existing load balancer** section, configure the following options:
      
        * Choose **Choose from your load balancer target groups**.
        * From the **Existing load balancer target groups** dropdown list, choose 
 **webserver-app | HTTP**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/56d27bbe-03ac-4cb8-9b7e-7eae631e9a7b)

*  In the **Health checks** section, under **Additional health check types**, select  :heavy_check_mark: Turn on **Elastic Load Balancing health checks**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5813f7e7-6e69-40ef-b143-d227357c1b21)

* **Step 53:** Choose **Next**.

* **Step 54:** On the **Configure group size and scaling policies – optional** page, configure the following options: 

   * In the **Group size – optional** section, enter the following values: 

     * **Desired capacity**: `2`

     * **Minimum capacity**: `2`

     * **Maximum capacity**: `4`

    * In the **Scaling policies – optional** section, configure the following options:

       * Choose  **Target tracking scaling policy**.
 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3574f67c-2464-4275-b900-1cce6e8aae55)
  
*  For **Metric type**, choose **Average CPU utilization**.
  *  For **Target value**, enter `50`
         





This change tells auto scaling to maintain an average CPU utilization across all instances of 50 percent. Auto scaling automatically adds or removes capacity as required to keep the metric at or close to the specified target value. It adjusts to fluctuations in the metric due to a fluctuating load pattern.

* **Step 55:** Choose **Next**.


* **Step 56:** On the ***Add notifications – optional** page, choose **Next**.
 
* **Step 57:** On the **Add tags – optional** page, choose **Add tag** and configure the following options:

   * For **Key**, enter Name

   * For **Value - optional**, enter WebApp
     
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3ec8bc48-bddf-490c-8570-2e1e49a7b182)

* **Step 58:** Choose **Next**.

* **Step 59:** On the **Review** page, choose **Create Auto Scaling group**.

These options launch EC2 instances in private subnets across both Availability Zones.

My Auto Scaling group initially shows an Instances count of zero, but new instances will be launched to reach the desired count of two instances.

**Note:** If you experience an error related to the t3.micro instance type not being available, then rerun this task by choosing the t2.micro instance type instead.


# Verifying the auto scaling configuration

 I verify that both the auto scaling configuration and the load balancer are working by accessing a pre-installed script on one of my servers that will consume CPU cycles, which invokes the scale out alarm.

* **Step 60:** In the left navigation pane, choose **Instances**.

Two new instances named **WebApp** are being created as part of my Auto Scaling group. While these instances are being created, the **Status check** for these two instances is Initializing.

Observe the **Status check** field for the instances until the status is 2/2 checks passed. Wait for the two new instances to complete initialization before I proceed to the next step. 

I might need to choose  **Refresh** to see the updated status.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/eeed0a99-a713-40a8-b800-cfe6cdb394b2)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/06fd5c1a-ca5d-4f1c-9e71-438dd7c8b21f)


* **Step 61:** Once the instances have completed initialization, in the left navigation pane in the Load Balancing section, choose Target Groups, and then select my target group, webserver-app.

* **Step 62:** On the **Targets** tab, verify that two instances are being created. Refresh this list until the **Health status** of these instances changes to healthy.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/92339fc6-f039-4742-a85a-0bdfd6a7057b)

I can now test the web application by accessing it through the load balancer.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0d354d6d-8881-48bf-a3fc-9b5eddcb59fc)


   # Testing auto scaling configuration


* **Step 63:** Open a new web browser tab, and paste the **DNS name** of the load balancer that you copied earlier into the address bar, and press Enter.

* **Step 64:** On the web page, choose **Start Stress**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cb77edd9-4325-4e85-8127-faab82eb2981)


This step calls the application **stress** in the background, which causes the CPU utilization on the instance that serviced this request to spike to 100 percent.

* **Step 65:** On the EC2 Management console, in the left navigation pane in the **Auto Scaling** section, choose **Auto Scaling Groups**.

* **Step 66:** Select  **Web App Auto Scaling Group**.

* **Step 67:** Choose the **Activity** tab. 

After a few minutes, I should see my Auto Scaling group add a new instance. This occurs because Amazon CloudWatch detected that the average CPU utilization of my Auto Scaling group exceeded 50 percent, and my scale-up policy has been invoked in response.

I can also check the new instances being launched on the EC2 Dashboard.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d2acf92a-5837-4bea-ab6f-1a4547517565)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c1775370-dcd9-43e7-b5a7-63e01a84da86)





