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

     * CafeInstanceURL	:52.38.0.14/cafe
* **Step 2:** Copy the **CafeInstanceURL** value, and paste it into a new browser window.

  **Note:** The **CafeinstanceURL** value looks similar to 34.55.102.33/cafe. 

* **Step 3:** Copy the other values from the table, and paste them into a text editor to use throughout the lab.

   * CafeInstancePublicDNS	: ec2-52-38-0-14.us-west-2.compute.amazonaws.com
   * SecretKey	: XDsGcPpNOrsueyMHf4LexykjTV0/8Ikkd8gbkuQd
   * CafeInstanceAZ	:  ${\color{magenta}us-west-2a}$
   * LabRegion	:us-west-2
   * CafeVpcID	: ${\color{orange}vpc-016cfa33748d28ba2}$
   * AccessKey	:AKIA2UC3F2GUQSB7QGMX
   * CafeSecurityGroupID	: ${\color{blue}sg-0d8539469600b38b5}$
     
* **Step 4:** On the cafe website, choose **Menu**, add at least one of each item to your order, and then choose **Submit Order**. 

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c27399fb-627d-43c1-a540-03c5736eb3d3)


* **Step 5:** Go to the **Order History** page, and record the number of orders that I placed. Later in this lab, I can compare this number with the number of orders in the migrated database.


# Creating an Amazon RDS instance by using the AWS CLI


I created an Amazon RDS instance by using the AWS CLI. To begin, I use EC2 Instance Connect to securely connect to the **CLI Host** instance already provisioned for me. This instance has the AWS CLI installed on it as part of provisioning. I run AWS CLI commands to do the following:

 * Configure the AWS CLI.

 * Create the following prerequisite components required to build the Amazon RDS instance:

     * A security group firewall for the Amazon RDS instance

     * Two private subnets and a database subnet group

  * Create the Amazon RDS MariaDB instance.
## Connecting to the CLI Host instance

 I use EC2 Instance Connect to connect to the CLI Host EC2 instance. I use this instance to run AWS CLI commands.

* **Step 6:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

* **Step 7:** In the navigation pane, choose **Instances**.

* **Step 8:** From the list of instances, select the :ballot_box_with_check: **CLI Host** instance.

  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/22280405-8029-4edd-a1cd-c4adc371ce2c)


* **Step 9:** Choose **Connect**.

* **Step 10:** On the **EC2 Instance Connect** tab, choose **Connect**.

  I  connected to the CLI Host instance, I can configure and use the AWS CLI to call AWS services.

# Configuring the AWS CLI

I configured the AWS CLI by providing the configuration parameters that were made available to me when the lab was provisioned. After configuration, I run AWS CLI commands to interact with AWS services.

* **Step 11:** To set up the AWS CLI profile with credentials, in the EC2 Instance Connect terminal, run the following command:

                aws configure

 * **Step 12:** When prompted, enter the following information:

**Note:** Use the values that I copied into the text editor earlier in the lab, and paste them into the terminal window prompt.

   * **AWS Access Key ID**: Enter the value for **AccessKey**(AKIA2UC3F2GUQSB7QGMX).

   * **AWS Secret Access Key:** Enter the value for **SecretKey**(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).

   * **Default region name:** Enter the value for **LabRegion**(us-west-2).
  
   * **Default output format:** Enter `json`.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7fa5c913-5d59-465e-9593-125db9ea24fa)


## Creating prerequisite components

I create the prerequisite infrastructure components for the Amazon RDS instance. Specifically, I create the following components that are shown in the final architecture diagram:

  * CafeDatabaseSG (Security group for the Amazon RDS database)

  * CafeDB Private Subnet 1

  * CafeDB Private Subnet 2

  * CafeDB Subnet Group (Database subnet group)

I run AWS CLI commands in the EC2 Instance Connect terminal.

I create the CafeDatabaseSG security group. This security group is used to protect the Amazon RDS instance. It will have an inbound rule that allows only MySQL requests (using the default TCP protocol and port 3306) from instances that are associated with the CafeSecurityGroup. This rule ensures that only the CafeInstance is able to access the database. 


 * **Step 12:** To create the security group, run the following command. In the command, replace **<CafeInstance VPC ID>** with the **CafeVpcID** value that I recorded earlier:
 
  
                  aws ec2 create-security-group \
                  --group-name CafeDatabaseSG \
                  --description "Security group for Cafe database" \
                  --vpc-id <CafeInstance VPC ID>

**Note**: Go to the top of these instructions, choose <code style="color : name_color">Details</code>, and then choose <code style="color : name_color">Show</code>.

 **CafeVpcID**	: ${\color{orange}vpc-016cfa33748d28ba2}$

                    aws ec2 create-security-group \
                    --group-name CafeDatabaseSG \
                    --description "Security group for Cafe database" \
                    --vpc-id vpc-016cfa33748d28ba2

   
   After run the command I got this output:
   
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/222ef670-5b9f-498d-9376-d074ed5ab766)

After the command completes, copy and paste the **GroupId** from the output to a text editor to use later. I use this value as the  ${\color{red}group ID:sg-0337d6b6f9d1b223e}$ for the **CafeDatabaseSG Group ID** in this lab. 

I created the inbound rule for the security group.

* **Step 13:** To create the inbound rule, run the following command. In the command, replace <CafeDatabaseSG Group ID> with the **GroupId** value that I recorded in the previous step, and replace <CafeSecurityGroup Group ID> with the **CafeSecurityGroupID** value that you recorded earlier in the lab:

            aws ec2 authorize-security-group-ingress \
            --group-id <CafeDatabaseSG Group ID> \
            --protocol tcp --port 3306 \
            --source-group <CafeSecurityGroup Group ID>
  
* ${\color{red}group ID:sg-0337d6b6f9d1b223e}$

**Note**: Go to the top of these instructions, choose <code style="color : name_color">Details</code>, and then choose <code style="color : name_color">Show</code>.
* CafeSecurityGroupID	: ${\color{blue}sg-0d8539469600b38b5}$

              aws ec2 authorize-security-group-ingress \
              --group-id sg-0337d6b6f9d1b223e \
              --protocol tcp --port 3306 \
              --source-group sg-0d8539469600b38b5

  After run the command I got this output:
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/694c1a02-8770-49bc-b85e-30e2f5e1039c)

**Note:** follow the colors.

* **Step 14:** To confirm that the inbound rule was applied appropriately, run the following command:

             aws ec2 describe-security-groups \
             --query "SecurityGroups[*].[GroupName,GroupId,IpPermissions]" \
             --filters "Name=group-name,Values='CafeDatabaseSG'"
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/95976a08-b2ef-433b-bd72-bc3ab5b69b4f)

The output of the command should show that the CafeDatabaseSG security group now has an inbound rule that allows connections from TCP port 3306 if the source of the connection is an instance that has the CafeSecurityGroup Group ID association.


I create two private subnets and a database subnet group. First, I create CafeDB Private Subnet 1. This subnet hosts the RDS DB instance. It is a private subnet that is defined in the same Availability Zone as the CafeInstance.

I must assign the subnet a Classless Inter-Domain Routing (CIDR) address block that is within the address range of the VPC but that does not overlap with the address range of any other subnet in the VPC. This reason is why I collected the information about the VPC and existing subnet CIDR blocks:
 
  * Cafe VPC IPv4 CIDR block: 10.200.0.0/20

  * Cafe Public Subnet 1 IPv4 CIDR block: 10.200.0.0/24
 
Consider these address ranges. Can I find a suitable CIDR block for the private subnet? One possible answer is to use the address range 10.200.2.0/23.

* **Step 15:** To create the subnet, run the following command. In the command, replace <CafeInstance VPC ID> and <CafeInstance Availability Zone> with the values of **CafeVpcID** and **CafeInstanceAZ**, respectively, that I recorded earlier.

       aws ec2 create-subnet \
       --vpc-id <CafeInstance VPC ID> \
       --cidr-block 10.200.2.0/23 \
       --availability-zone <CafeInstance Availability Zone>


 **CafeVpcID**	: ${\color{orange}vpc-016cfa33748d28ba2}$
 
 **CafeInstanceAZ** :  ${\color{magenta}us-west-2a}$

       aws ec2 create-subnet \
      --vpc-id vpc-016cfa33748d28ba2 \
      --cidr-block 10.200.2.0/23 \
       --availability-zone us-west-2a
 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/905eec52-aa8f-4c93-96b0-74973d2e579c)

From the output of the command, note the value for **SubnetId:** ${\color{cyan}subnet-08b67929eebffe7a2}$. You use this information later for CafeDB **Private Subnet 1**.


I create CafeDB Private Subnet 2. This is the extra subnet that is required to form the database subnet group. It is an empty private subnet that is defined in a different Availability Zone than the CafeInstance.

Similar to what I did in the previous steps, I must assign a CIDR address block to the subnet that is within the address range of the VPC but does not overlap with the address range of any other subnet in the VPC. So far, I have used the following address ranges:

 * Cafe VPC IPv4 CIDR block: 10.200.0.0/20

 * Cafe Public Subnet 1 IPv4 CIDR block: 10.200.0.0/24

 * Cafe Private Subnet 1 IPv4 CIDR block: 10.200.2.0/23

I use the 10.200.10.0/23 address range for this second private subnet.

For the Availability Zone for the second subnet, I can choose any other Availability Zone (but not the one ending in a).



* **Step 16:** To create the second subnet, run the following command. In the command, replace <CafeInstance VPC ID> with the value of **CafeVpcID** that I recorded earlier, and replace <availability-zone> with an Availability Zone that is different than the one that you used for the first subnet (for example, **us-west-2b**).

      aws ec2 create-subnet \
      --vpc-id <CafeInstance VPC ID> \
      --cidr-block 10.200.10.0/23 \
      --availability-zone <availability-zone>

 **CafeVpcID**	: ${\color{orange}vpc-016cfa33748d28ba2}$
 
 **Availability-zone** :  us-west-2b


          aws ec2 create-subnet \
         --vpc-id vpc-016cfa33748d28ba2 \
         --cidr-block 10.200.10.0/23 \
         --availability-zone us-west-2b

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/76fb5da7-4b21-4213-b045-f70ead541397)

From the output of the command, note the value for **SubnetId=**  ${\color{fuchsia}subnet-00ab79cf373f2bd76}$. I use this information later for CafeDB Private **Subnet 2**.

Next, I create CafeDB Subnet Group. 

For the Amazon RDS instance for the café, the DB subnet group consists of the two private subnets that I created in the previous steps: CafeDB Private Subnet 1 and CafeDB Private Subnet 2.


* **Step 17:** In the terminal window, run the following command. In the command, replace <Cafe Private Subnet 1 ID> and <Cafe Private Subnet 2 ID> with the subnet ID values that you recorded earlier for each private subnet. Note that there is a space between the subnet IDs in the command.

       aws rds create-db-subnet-group \
       --db-subnet-group-name "CafeDB Subnet Group" \
       --db-subnet-group-description "DB subnet group for Cafe" \
       --subnet-ids <Cafe Private Subnet 1 ID> <Cafe Private Subnet 2 ID> \
       --tags "Key=Name,Value= CafeDatabaseSubnetGroup"

**Prvate SubnetId 1:** ${\color{cyan}subnet-08b67929eebffe7a2}$

**Private SubnetId 2:** ${\color{fuchsia}subnet-00ab79cf373f2bd76}$


        aws rds create-db-subnet-group \
        --db-subnet-group-name "CafeDB Subnet Group" \
        --db-subnet-group-description "DB subnet group for Cafe" \
        --subnet-ids subnet-08b67929eebffe7a2 subnet-00ab79cf373f2bd76 \
        --tags "Key=Name,Value= CafeDatabaseSubnetGroup"
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5dde4d20-1e56-4d7d-b5ba-ff95322d73ae)

After the command completes, it returns the attributes of the DB subnet group.

## Creating the Amazon RDS MariaDB instance

I create the CafeDBInstance that is shown in the final architecture. Using the AWS CLI, I create an Amazon RDS MariaDB instance with the following configuration settings:

* DB instance identifier: CafeDBInstance

* Engine option: MariaDB

* DB engine version: 10.5.18

* DB instance class: db.t3.micro

* Allocated storage: 20 GB

* Availability Zone: CafeInstanceAZ

* DB Subnet group: CafeDB Subnet Group

* VPC security groups: CafeDatabaseSG

* Public accessibility: No

* Username: root

* Password: Re:Start!9

  These options specify the creation of a MariaDB database instance that is deployed in the same Availability Zone as the café instance. The MariaDB database instance also uses the DB subnet group that I built in the previous step.


  * **Step 18:** In the terminal window, run the following command. In the command, replace <CafeInstance Availability Zone> with the **CafeInstanceAZ** value that you recorded at the beginning of the lab, and replace <CafeDatabaseSG Group ID> with the value that you recorded in a previous step.

        aws rds create-db-instance \
        --db-instance-identifier CafeDBInstance \
        --engine mariadb \
        --engine-version 10.5.18 \
        --db-instance-class db.t3.micro \
        --allocated-storage 20 \
        --availability-zone <CafeInstance Availability Zone> \
        --db-subnet-group-name "CafeDB Subnet Group" \
        --vpc-security-group-ids <CafeDatabaseSG Group ID> \
        --no-publicly-accessible \
        --master-username root --master-user-password 'Re:Start!9'


  **Note:** Here just be careful about engine version I use **engine-version 10.5.18**


         aws rds create-db-instance \
        --db-instance-identifier CafeDBInstance \
        --engine mariadb \
        --engine-version 10.5.18 \
        --db-instance-class db.t3.micro \
        --allocated-storage 20 \
        --availability-zone us-west-2a \
        --db-subnet-group-name "CafeDB Subnet Group" \
        --vpc-security-group-ids sg-0337d6b6f9d1b223e \
        --no-publicly-accessible \
        --master-username root --master-user-password 'Re:Start!9'

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/87224fc2-54e4-4081-a944-852f5600cd24)

  The command immediately returns some information about the database, but the database instance might take up to 10 minutes to become available.


* **Step 19:** To check the status of the database, run the following command:
   I monitor the status of the database instance until it shows a status of **available**.


       aws rds describe-db-instances \
      --db-instance-identifier CafeDBInstance \
      --query "DBInstances[*]. 
      [Endpoint.Address,AvailabilityZone,PreferredBackupWindow,BackupRetentionPeriod,DBInstanceStatus]"
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/92a74ed8-9e5c-491e-ad10-d8e85c1facad)

Keep repeating the command until the status shows available.

       RDS Instance Database Endpoint Address: cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com

# Migrating application data to the Amazon RDS instance

I migrate the data from the existing local database to the newly created Amazon RDS database. Specifically, I do the following:

Connect to the **CafeInstance** by using EC2 Instance Connect.

Use the mysqldump utility to create a backup of the local database.

Restore the backup to the Amazon RDS database.

Test the data migration.

Note that you perform these steps from the command line after connecting to the CafeInstance. This instance can communicate with the Amazon RDS instance by using the MySQL protocol because you associated the CafeDatabaseSG security group with the Amazon RDS instance.


* **Step 20:** Connect to the **CafeInstance** by using EC2 Instance Connect. Follow the instructions that you used earlier to connect to the CLI Host instance.
  
 I use the mysqldump utility to create a backup of the local cafe_db database. This utility program is part of the MySQL database product, and it is available for me to use. 

  *  On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management 
  Console**.

  * In the navigation pane, choose **Instances**.

  * From the list of instances, select the :ballot_box_with_check:**CafeInstance**.

  * Choose **Connect**.

  * On the **EC2 Instance Connect** tab, choose **Connect**.

* **Step 21:** In the terminal window, run the following command:

        mysqldump --user=root --password='Re:Start!9' \
        --databases cafe_db --add-drop-database > cafedb-backup.sql
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f703a527-3617-41dd-ae2f-dfbf666153e4)


This command generates SQL statements in a file named cafedb-backup.sql, which can be run to reproduce the schema and data of the original cafe_db database.

* **Step 22:** To review the contents of the backup file, open the cafedb-backup.sql file in my preferred text editor. Alternatively, if I want to view it using the Linux less command, enter the following command in the terminal window:

                  less cafedb-backup.sql

  **Tip:** When I use the less command, use the up and down arrow keys to move one line up or one line down, respectively. I can also use the Page up and Page down keys to navigate one page up or one page down, respectively. To quit the command, enter **q**.


  Notice the various SQL commands that create the database, create its tables and indexes, and populate the database with its original data.

I restore the backup to the Amazon RDS database by using the mysql command. I must specify the endpoint address of the Amazon RDS instance in the command. 

* **Step 23:** In the terminal window, enter the following command. In the command, replace <RDS Instance Database Endpoint Address> with the value that I recorded earlier.

           mysql --user=root --password='Re:Start!9' \
           --host=<RDS Instance Database Endpoint Address> \
           < cafedb-backup.sql

 RDS Instance Database Endpoint Address: **cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com** 

 
          mysql --user=root --password='Re:Start!9' \
          --host=cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com \
          < cafedb-backup.sql

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/05c7bfc4-daea-46f3-834d-84aa3710c124)

This command creates a MySQL connection to the Amazon RDS instance and runs the SQL statements in the cafedb-backup.sql file.

Finally, I verify that the cafe_db was successfully created and populated in the Amazon RDS instance. I open an interactive MySQL session to the instance and retrieve the data in the product table of the cafe_db database. 

* **Step 24:** In the SSH window, enter the following command. In the command, replace <RDS Instance Database Endpoint Address> with the value that you recorded earlier.
  
RDS Instance Database Endpoint Address: **cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com** 

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a2d6c9e5-958b-4753-9198-242c64a6e4ea)

 * **Step 25:** Next, enter the SQL statement to retrieve the data in the product table:

           select * from product;

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/88a18b06-deb0-41f4-b54b-43c4e8153ccd)
    The query should return rows from the table. Ensure that the information matches the number of items that I ordered at the beginning of the lab.

I can now exit the interactive SQL session. 

 * **Step 26:** In the terminal window, enter the following command:

         exit

# Configuring the website to use the Amazon RDS instance

I ready to configure the café website to use the Amazon RDS instance. This step is straightforward because the designer of the application followed best practices and externalized the database connection information as parameters in Parameter Store, a capability of AWS Systems Manager. In this task, I change the database URL parameter of the café application to point to the endpoint address of the RDS instance.

* **Step 27:** On the **AWS Management Console**, in the **Search** bar, enter and choose `Systems Manager` to go to **AWS Systems Manager**.

* **Step 28:** In the left navigation pane, choose **Parameter Store**.
* 
* **Step 29:** From the **My parameters list**, choose **/cafe/dbUrl**. The current value of the parameter is displayed, along with its description and other metadata information.

* **Step 30:** Choose **Edit**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9dcdfa1a-c97a-4406-8764-5031dea1c68c)

* **Step 31:** In the **Parameter details** page, replace the text in the **Value** box with the **RDS Instance Database Endpoint Address** value that you recorded earlier.
  
In parameter details only change **value box** = cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com rest of thing are default.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/369d02ab-43d1-4841-94c0-ea1fdede334d)

 * **Step 32:** Choose **Save changes**.
The dbUrl parameter now references the RDS DB instance instead of the local database.

I test the website to confirm that it is able to access the new database correctly.

* **Step 33:** In a new browser window, paste the URL for **CafeInstanceURL** that I copied to a text editor at the beginning of the lab.
  
   **CafeInstanceURL :52.38.0.14/cafe**

  The website's home page should load correctly.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0884c2bd-cb03-4330-9fad-d29cf7b84a3a)


* **Step 34:** Choose the Order History tab. and observe the number of orders in the database. Compare this number with the number of orders that I recorded before the database migration. Both numbers should be the same.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f10bf04d-6600-4d5a-8328-c95c419c5bfc)

# Monitoring the Amazon RDS database

One of the benefits of using Amazon RDS is the ability to monitor the performance of a database instance. Amazon RDS automatically sends metrics to CloudWatch every minute for each active database. In this task, I identify some of these performance metrics and learn how to monitor a metric in the Amazon RDS console.

* **Step 35:** On the **AWS Management Console**, in the **Search** bar, enter and choose RDS to open the **RDS Management Console**.

* **Step 36:** In the left navigation pane, choose **Databases**.

* **Step 37:**  From the **Databases** list, choose **cafedbinstance**. Detailed information about the database is displayed.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/55a33ed0-fec9-4ad0-b997-28baae6bf54d)



* **Step 37:** Choose the **Monitoring** tab. By default, this tab displays a number of key database instance metrics that are available from CloudWatch. Each metric includes a graph that shows the metric as it is monitored over a specific time span.

The list of displayed metrics includes the following:

 * **CPUUtilization:** The percent of CPU utilization

 * **DatabaseConnections:** The number of database connections in use

 * **FreeStorageSpace:** The amount of available storage space

 * **FreeableMemory:** The amount of memory (RAM) available on the Amazon RDS instance

 * **WriteIOPS:** The average number of disk write I/O operations per second

 * **ReadIOPS:** The average number of disk read I/O operations per second

**Tip:** Some of the metrics listed might appear on the second or third pages of charts. To see additional metrics, choose 2 or 3 to the right of the CloudWatch search box.

I monitor the **DatabaseConnections** metric as I create a connection to the database from the **CafeInstance**.



* **Step 38:** To open an interactive SQL session to the RDS cafe_db instance, in the CafeInstance terminal window, enter the following command. In the command, replace <RDS Instance Database Endpoint Address> with the value that you recorded earlier.

              mysql --user=root --password='Re:Start!9' \
              --host=<RDS Instance Database Endpoint Address> \
              cafe_db

  
* RDS Instance Database Endpoint Address: **cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com**

            mysql --user=root --password='Re:Start!9' \
            --host=cafedbinstance.c9ycyqekqles.us-west-2.rds.amazonaws.com \
            cafe_db

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/68cb2fc8-b2e5-4868-abb2-0168b1a9703f)

 * **Step 39:** To retrieve the data in the product table, enter the following SQL statement:

                   select * from product;

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d3d01ec5-af02-4257-b65d-914ca00caddf)


 * **Step 40:** In the Amazon RDS console, choose the **DatabaseConnections** graph to open it in a larger view. The graph shows a line that indicates that **1** connection is in use. This connection was established by the interactive SQL session from the CafeInstance.

**Tip:** If the graph does not show any connections in use, wait 1 minute (which is the sampling interval), and then choose Refresh. It should now show one open connection.


 * **Step 41:** To close the connection from the interactive SQL session, in the CafeInstance terminal window, enter the following command:

                exit

   
 * **Step 42:** Wait 1 minute, and then in the DatabaseConnections graph in Amazon RDS console, choose Refresh. The graph now shows that the number of connections in use is 0.


## Conclusion
I have successfully done the following:

* Created an Amazon RDS MariaDB instance by using the AWS CLI

* Migrated data from a MariaDB database on an EC2 instance to an Amazon RDS MariaDB instance

* Monitored the Amazon RDS instance by using CloudWatch metrics









