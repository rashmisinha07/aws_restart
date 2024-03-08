# Migrating to Amazon RDS

## overview

I migrated the café web application to use a fully managed Amazon Relational Database Service (Amazon RDS) database (DB) instance instead of a local database instance.

I begin by generating some data on the existing database. This data is migrated to the new Amazon RDS instance.

During the migration process, I build the required components, including two private subnets in different Availability Zones, a security group for the database instance, and the RDS DB instance itself. After the database has been migrated, I reconfigure the café application to use the Amazon RDS instance instead of a local database.

## Starting architecture

The following diagram illustrates the topology of the café web application runtime environment before the migration. The application database runs in an Amazon Elastic Compute Cloud (Amazon EC2) Linux, Apache, MySQL, and PHP (LAMP) instance along with the application code. The instance has a T3 small instance type and runs in a public subnet so that internet clients can access the website. A CLI Host instance resides in the same subnet to facilitate the administration of the instance by using the AWS Command Line Interface (AWS CLI).


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/26004d5d-354e-4160-a767-205ae456c03f)


Final architecture

The following diagram illustrates the topology of the café web application runtime environment after the migration.

I migrate the local café database to an Amazon RDS database that resides outside the instance. The Amazon RDS database is deployed in the same virtual private cloud (VPC) as the instance.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/50c9569b-05e3-4c90-8c79-a64e168a13b0)

# Task 1: Generating order data on the café website

I browse the café website and place a few orders that are stored in the existing database. Placing orders creates data for the application before the application is migrated to new Amazon RDS instance.

I open the café web application, and place some orders.

* **Step 1:** To place orders, these are the details:
  
* CafeInstancePublicDNS	: ec2-52-38-0-14.us-west-2.compute.amazonaws.com
* SecretKey	: XDsGcPpNOrsueyMHf4LexykjTV0/8Ikkd8gbkuQd
* CafeInstanceAZ	: us-west-2a
* LabRegion	:us-west-2
* CafeVpcID	:vpc-016cfa33748d28ba2
* AccessKey	:AKIA2UC3F2GUQSB7QGMX
* CafeSecurityGroupID	:sg-0d8539469600b38b5
* CafeInstanceURL	:52.38.0.14/cafe
Copy the CafeInstanceURL value, and paste it into a new browser window.

Note: The CafeinstanceURL value looks similar to 34.55.102.33/cafe. 

Copy the other values from the table, and paste them into a text editor to use throughout the lab.

On the cafe website, choose Menu, add at least one of each item to your order, and then choose Submit Order.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c27399fb-627d-43c1-a540-03c5736eb3d3)

Go to the Order History page, and record the number of orders that you placed. Later in this lab, you can compare this number with the number of orders in the migrated database.


# Creating an Amazon RDS instance by using the AWS CLI


you create an Amazon RDS instance by using the AWS CLI. To begin, you use EC2 Instance Connect to securely connect to the CLI Host instance already provisioned for you. This instance has the AWS CLI installed on it as part of provisioning. You then run AWS CLI commands to do the following:

Configure the AWS CLI.

Create the following prerequisite components required to build the Amazon RDS instance:

A security group firewall for the Amazon RDS instance

Two private subnets and a database subnet group

Create the Amazon RDS MariaDB instance.
# Connecting to the CLI Host instance

 you use EC2 Instance Connect to connect to the CLI Host EC2 instance. You use this instance to run AWS CLI commands.

On the AWS Management Console, in the Search bar, enter and choose EC2 to open the EC2 Management Console.

In the navigation pane, choose Instances.

From the list of instances, select the  CLI Host instance.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/22280405-8029-4edd-a1cd-c4adc371ce2c)
Choose Connect.

On the EC2 Instance Connect tab, choose Connect.

Note: If you prefer to use an SSH client to connect to the EC2 instance, see the guidance to Connect to Your Linux Instance.

Now that you are connected to the CLI Host instance, you can configure and use the AWS CLI to call AWS services.

# Configuring the AWS CLI

you configure the AWS CLI by providing the configuration parameters that were made available to you when the lab was provisioned. After configuration, you run AWS CLI commands to interact with AWS services.

To set up the AWS CLI profile with credentials, in the EC2 Instance Connect terminal, run the following command:
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7fa5c913-5d59-465e-9593-125db9ea24fa)


# Creating prerequisite components

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/222ef670-5b9f-498d-9376-d074ed5ab766)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/694c1a02-8770-49bc-b85e-30e2f5e1039c)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/95976a08-b2ef-433b-bd72-bc3ab5b69b4f)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/905eec52-aa8f-4c93-96b0-74973d2e579c)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/76fb5da7-4b21-4213-b045-f70ead541397)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5dde4d20-1e56-4d7d-b5ba-ff95322d73ae)

# Creating the Amazon RDS MariaDB instance
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/92a74ed8-9e5c-491e-ad10-d8e85c1facad)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/87224fc2-54e4-4081-a944-852f5600cd24)

34. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a2d6c9e5-958b-4753-9198-242c64a6e4ea)

35. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/88a18b06-deb0-41f4-b54b-43c4e8153ccd)

# Configuring the website to use the Amazon RDS instance

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9dcdfa1a-c97a-4406-8764-5031dea1c68c)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/369d02ab-43d1-4841-94c0-ea1fdede334d)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0884c2bd-cb03-4330-9fad-d29cf7b84a3a)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f10bf04d-6600-4d5a-8328-c95c419c5bfc)

# Monitoring the Amazon RDS database

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/55a33ed0-fec9-4ad0-b997-28baae6bf54d)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/68cb2fc8-b2e5-4868-abb2-0168b1a9703f)

51![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d3d01ec5-af02-4257-b65d-914ca00caddf)







