# Working with AWS Lambda
## overview

I deployed and configure an AWS Lambda based serverless computing solution. The Lambda function generates a sales analysis report by pulling data from a database and emailing the results daily. The database connection information is stored in Parameter Store, a capability of AWS Systems Manager. The database itself runs on an Amazon Elastic Compute Cloud (Amazon EC2) Linux, Apache, MySQL, and PHP (LAMP) instance.

The following diagram shows the architecture of the sales analysis report solution and illustrates the order in which actions occur.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f938693e-d731-45a1-a3af-f246a8963d9b)

The diagram includes the following function steps:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e4a1e04c-367d-4c6a-a790-1e01edc5e15f)

# Observing the IAM role settings

I  create two Lambda functions. Each function requires permissions to access the AWS resources with which they interact. 

I analyze the IAM roles and the permissions that they grant to the salesAnalysisReport and salesAnalysisReportDataExtractor Lambda functions that I create later.

## Observing the salesAnalysisReport IAM role settings

* **Step 1:** In the AWS Management Console, choose **Services > Security, Identity, & Compliance > IAM**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/817ec034-7264-4081-ba0c-745c4c5c2798)

* **Step 2:** In the navigation pane, choose **Roles**.

* **Step 3:** In the search box, enter `sales`

* **Step 4:** From the filtered results, choose the **salesAnalysisReportRole** hyperlink.
  
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f16f59a9-f814-4bb9-aa35-a4416343e11b)

* **Step 5:** Choose the **Trust relationships** tab, and notice that lambda.amazonaws.com is listed as a trusted entity, which means that the Lambda service can use this role.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/51942946-8843-4042-b6d3-90f323e8d710)

* **Step 6:** Choose the **Permissions** tab, and notice the four policies assigned to this role. To expand each role and analyze the permissions that each policy grants, choose the **+** icon next to each role:
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a8b42bb5-7756-4841-872b-805a2244c6db)

  * **AmazonSNSFullAccess** provides full access to Amazon SNS resources.
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fe4c7579-70e4-45db-aa17-4844869b1fc2)

  * **AmazonSSMReadOnlyAccess** provides read-only access to Systems Manager resources.
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/055141f3-d156-4cf7-bdda-6613272be231)

  * **AWSLambdaBasicRunRole** provides write permissions to CloudWatch logs (which are required by every Lambda function).
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4438311a-96e2-4797-9ffc-52241570fd8c)

   * **AWSLambdaRole**  gives a Lambda function the ability to invoke another Lambda function.
     
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6610b7d2-c190-4d61-b7f6-7fdcd8eafff7)

The salesAnalysisReport Lambda function that you create later in this lab uses the salesAnalysisReportRole role.


## Observing the salesAnalysisReportDERole IAM role settings

* **Step 7:** Choose **Roles** again.

* **Step 8:** In the search box, enter `sales`

* **Step 9:** From the filtered results, choose the **salesAnalysisReportDERole** hyperlink.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/94d27166-d00d-4e9e-a162-d4120fd17022)

* **Step 10:** Choose the **Trust relationships** tab, and notice that lambda.amazonaws.com is listed as a trusted entity.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ab34bd3f-c45d-485e-9f74-43b4906e2ad7)

* **Step 11:** Choose the **Permissions** tab, and notice the permissions granted to this role:

   * **AWSLambdaBasicRunRole** provides write permissions to CloudWatch logs.
     
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4f9cdc18-b374-445d-bf13-f411fe8bf2af)

  * **AWSLambdaVPCAccessRunRole** provides permissions to manage elastic network interfaces to connect a function to a virtual private cloud (VPC).
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ac981bd1-7399-4444-a98f-ac4595860353)

The salesAnalysisReportDataExtractor Lambda function that you create next uses the salesAnalysisReportDERole role.

# Creating a Lambda layer and a data extractor Lambda function

 I first create a Lambda layer, and then I create a Lambda function that uses the layer.

Start by downloading two required files.

* **Step 12:** To download the lab files required by this task to my local machine, choose the following links:

   pymysql-v3.zip

   salesAnalysisReportDataExtractor-v3.zip

**Note:** The salesAnalysisReportDataExtractor-v3.zip file is a Python implementation of a Lambda function that makes use of the PyMySQL open-source client library to access the MySQL café database. This library has been packaged into the pymysql-v3.zip which is uploaded to Lambda layer next.

## Creating a Lambda Layer

I create a Lambda layer named pymysqlLibrary and upload the client library into it so that it can be used by any function that requires it. Lambda layers provide a flexible mechanism to reuse code between functions so that the code does not have to be included in each function’s deployment package.

* **Step 13:** In the AWS Management Console, choose **Services > Compute > Lambda**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a83641a0-f197-4df1-926e-177cc7d07108)

**Tip:** If the navigation panel is closed, choose the collapsed menu icon (three horizontal lines) to open the **AWS Lambda** panel.

* **Step 14:** Choose **Layers**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/33c8de92-f264-4f09-a61a-86b502a09ea3)

* **Step 15:** Choose **Create layer**.

* **Step 16:** Configure the following layer settings:

   * For **Name**, enter `pymysqlLibrary`

   * For **Description**, enter `PyMySQL library modules`

   * Select **Upload a .zip file**. To upload the pymysql-v3.zip file, choose **Upload**, navigate to the folder where you downloaded the pymysql-v3.zip file, and open it.

   * For **Compatible runtimes**, choose **Python 3.9**.
     
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f7083171-b4a5-40a2-9c52-784f1f59f629)

* **Step 17:** Choose **Create**.

The message "Successfully created layer pymysqlLibrary version 1" is displayed.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e8a5eaf4-c577-4202-b4ea-4371c2cbbc2e)

**Tip:** The Lambda layers feature requires that the .zip file containing the code or library conform to a specific folder structure. The pymysqlLibary.zip file used in this lab was packaged using the following folder structure:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d4c6720a-1dbe-4bb2-93d1-b68d04611a52)

## Creating a data extractor Lambda function

* **Step 18:** In the navigation pane, choose **Functions** to open the **Functions** dashboard page.

 * **Step 19:** Choose **Create function**, and configure the following options:

   * At the top of the **Create function** page, select **Author from scratch**.

   * For **Function name**, enter `salesAnalysisReportDataExtractor`

   * For Runtime, choose Python 3.9.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9a9b62d9-4ba4-4f48-9edf-ae610caf7bf7)

   * Expand **Change default execution role**, and configure the following options:    

     * For **Execution role**, choose **Use an existing role**.

     * For **Existing role**:, choose **salesAnalysisReportDERole**.
       
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6aa8a19a-41ef-47e2-9198-ab0a5a57d246)

* **Step 20:** Choose **Create function**.

    A new page opens with the following message: "Successfully created the function salesAnalysisReportDataExtractor."
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/219f6f51-03a8-4370-bfca-90de53f9cd88)

## Adding the Lambda layer to the function

* **Step 21:** In the **Function overview** panel, choose **Layers**.

* **Step 22:** At the bottom of the page, in the **Layers** panel, choose **Add a layer**.

* **Step 23:** On the **Add layer** page, configure the following options:

  * **Choose a layer**: Choose **Custom layers**.

  * **Custom layers**: Choose **pymysqlLibrary**.

  * **Version**: Choose **1**.
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6e69e07a-f0f4-42e7-93fc-dba298db711e)

* **Step 24:** Choose **Add**.

The **Function overview** panel shows a count of **(1)** in the **Layers** node for the function.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/176de07f-2cfb-4733-ae9d-3f1e3ef93083)


## Importing the code for the data extractor Lambda function

* **Step 25:** Go to the **Lambda > Functions > salesAnalysisReportDataExtractor** page.

* **Step 26:** In the **Runtime settings** panel, choose **Edit**.

* **Step 27:** For **Handler**, enter `salesAnalysisReportDataExtractor.lambda_handler`
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4c77e33f-2eb4-4cf3-a23e-316be2ac8725)

* **Step 28:** Choose **Save**.

* **Step 29:** In the **Code source** panel, choose **Upload from**.

* **Step 30:** Choose **.zip file**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8c6dcfd8-0b4a-4629-8854-08d0edf1ca09)

* **Step 31:** Choose **Upload**, and then navigate to and select the **salesAnalysisReportDataExtractor-v3.zip** file that you downloaded earlier.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8fcfe5d2-bf2d-4297-bf3a-fcbbbdcb0bc7)

* **Step 32:** Choose **Save**.

The Lambda function code is imported and displays in the **Code source** panel. If necessary, in the **Environment** navigation pane, double-click **salesAnalysisReportDataExtractor.py** to display the code.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fc2d5c5d-7e9c-4cb3-950d-beb585d3c037)

* **Step 33:** Review the Python code that implements the function.

**Note:** If the code does not yet display in the function code editor, refresh the console so that it displays.

Read the comments included in the code to gain an understanding of its logic flow. Notice that the function expects to receive the database connection information (dbURL, dbName, dbUser, and dbPassword) in the event input parameter.

## Configuring network settings for the function

The final step before I can test the function is to configure its network settings. As the architecture diagram at the start of this lab shows, this function requires network access to the café database, which runs in an EC2 LAMP instance. Therefore, I need to specify the instance’s VPC, subnet, and security group information in the function’s configuration.

* **Step 34:** Choose the Configuration tab, and then choose VPC.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/818a484c-d30f-4b4d-9bba-9727ad2fc858)

* **Step 35:** Choose Edit, and configure the following options:

    * VPC: Choose the option with Cafe VPC as the Name.

    * Subnets: Choose the option with Cafe Public Subnet 1 as the Name.

     Tip: You can ignore the warning (if any) that recommends choosing at least two subnets to run in high availability mode because it is not applicable to the function.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e029223b-81d5-4753-a449-151f44719bdd)

    * Security groups: Choose the option with CafeSecurityGroup as the Name.

Notice that the security group’s inbound and outbound rules are automatically displayed following the field.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/296bf039-a010-4616-9fe4-b74135a0854b)

* **Step 36:** Choose Save.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/afaa8604-2f57-42a4-ae12-f4c5ac377a5e)

# Testing the data extractor Lambda function

## Launching a test of the Lambda function

You are now ready to test the salesAnalysisReportDataExtractor function. To invoke it, you need to supply values for the café database connection parameters. Recall that these are stored in Parameter Store.

* **Step 37:** On a new browser tab, open the AWS Management Console, and choose Services > Management & Governance > Systems Manager.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5354f385-0cbf-4413-9830-c0ef53a5abfd)

* **Step 38:** In the navigation pane, choose Parameter Store.

* **Step 39:** Choose each of the following parameter names, and copy and paste the Value of each one into a text editor document:

    * /cafe/dbUrl  :ec2-34-221-112-173.us-west-2.compute.amazonaws.com

   * /cafe/dbName : cafe_db

   * /cafe/dbUser : root

   * /cafe/dbPassword : Re:Start!9

* **Step 40:** Return to the Lambda Management Console browser tab. On the salesAnalysisReportDataExtractor function page, choose the Test tab.

* **Step 41:** Configure the Test event panel as follows:

   * For Test event action, select Create new event.

    * For Event name, enter SARDETestEvent

   * For Template, choose hello-world.

    * In the Event JSON pane, replace the JSON object with the following JSON object:

        {
          "dbUrl": "<value of /cafe/dbUrl parameter>",
          "dbName": "<value of /cafe/dbName parameter>",
         "dbUser": "<value of /cafe/dbUser parameter>",
         "dbPassword": "<value of /cafe/dbPassword parameter>"
         }

       * In this code, substitute the value of each parameter with the values that you pasted into a text editor in the previous steps. Enclose these values in quotation marks.
     
 * **Step 42:**  Choose Save.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1ad061c3-94f7-453c-a432-e8911efa90a7)

* **Step 43:** Choose Test.

After a few moments, the page shows the message "Execution result: failed". 

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/07a0d85e-cddb-4920-81a4-82ed492d299e)

## Troubleshooting the data extractor Lambda function


To open the café website on a new browser tab, find the public IP address of the café EC2 instance. 

The URL for the website has the format http://publicIP/cafe where publicIP is the public IP address of the café EC2 instance. There are two ways to find the public IP address:

Option 1: 

On the AWS Management Console, choose Services > Compute > EC2.

In the navigation pane, choose Instances.

Choose CafeInstance.

Copy the Public IPv4 address to a text editor. 

On a new browser tab, enter http://publicIP/cafe, and replace publicIP with the public IPv4 address that you just copied to a text editor.

Press Enter to load the café website. In the Execution result pane, choose Details to expand it, and notice that the error object returned a message similar to the following message after the function ran:


    {
     "errorMessage": "2019-02-14T04:14:15.282Z ff0c3e8f-1985-44a3-8022-519f883c8412 Task timed out after 
   3.00 seconds"
     }



  This message indicates that the function timed out after 3 seconds.

The Log output section includes lines starting with the following keywords:

   * START indicates that the function started running.

   * END indicates that the function finished running.

   * REPORT provides a summary of the performance and resource utilization statistics related to when the function ran.

What caused this error?

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/58ca919c-d682-41fc-a4c6-454d034e5af1)

## Analyzing and correcting the Lambda function

you analyze and correct the issue observed when you tested the Lambda function.

Here are a few hints to help you find the solution:

One of the first things that this function does is connect to the MySQL database running in a separate EC2 instance. It waits a certain amount of time to establish a successful connection. After this time passes, if the connection is unsuccessful, the function times out.

By default, a MySQL database uses the MySQL protocol and listens on port number 3306 for client access.

Choose the Configuration tab again, and choose VPC. Notice the Inbound rules for the security group that are used by the EC2 instance running the database. Is the database port number (3306) listed? You can choose the security group link if you want to edit and add an inbound rule to it.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8cf95094-6feb-4b5b-af77-22029d240893)

Once you have corrected the problem, return to browser tab with the salesAnalysisReportDataExtractor function page. Choose the Test tab, and choose Test again.

You should now see a green box showing the message “Execution result: succeeded (logs).” This message indicates that the function ran successfully.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0180a143-31c4-46f4-9876-c7b8600df9d5)

Choose Details to expand it.

The function returned the following JSON object:

   {
     "statusCode": 200,
     "body": []
    }

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/51c819b1-5e6b-4c4f-a9b2-41dcfdf8e537)

The body field, which contains the report data that the function extracted, is empty because there is no order data in the database.


## Placing an order and testing again

you access the café website and place some orders to populate data in the database. 

* **Step 11:** To open the café website on a new browser tab, find the public IP address of the café EC2 instance. 

The URL for the website has the format http://publicIP/cafe where publicIP is the public IP address of the café EC2 instance. There are two ways to find the public IP address:

Option 1: 

On the AWS Management Console, choose Services > Compute > EC2.

In the navigation pane, choose Instances.

Choose CafeInstance.

Copy the Public IPv4 address to a text editor. 34.221.112.173
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d49856ee-a8dc-4f7e-b45d-75a75bf1747d)

On a new browser tab, enter http://publicIP/cafe, and replace publicIP with the public IPv4 address that you just copied to a text editor.

Press Enter to load the café website.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0aafea10-dda9-419c-8222-b69bdb173de5)

Option 2:

At the top of these instruction, choose Details, and then choose Show. 

From the Credentials window, copy and paste the CafePublicIP to a text editor.

On a new browser tab, enter http://publicIP/cafe, and replace publicIP with the public IPv4 address that you just copied to a text editor.

Press Enter to load the café website.

* **Step 11:** On the café website, choose Menu, and place some orders to populate data in the database. 
 Now that there is order data in the database, you test the function again. 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/024db187-83e7-4e88-a53b-08480f00688f)

Go to the browser tab with the salesAnalysisReportDataExtractor function page.

Choose the Test tab, and choose Test.

The returned JSON object now contains product quantity information in the body field similar to the following:

{
  "statusCode": 200,
  "body": [
    {
      "product_group_number": 1,
      "product_group_name": "Pastries",
      "product_id": 1,
      "product_name": "Croissant",
      "quantity": 1
    },
    {
      "product_group_number": 2,
      "product_group_name": "Drinks",
      "product_id": 8,
      "product_name": "Hot Chocolate",
      "quantity": 2
     }


     ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cbc946dd-984b-4f42-a270-7a3698f704b2)


    # Configuring notifications

    ## Creating an SNS topic


    In this task, you create the SNS topic where the sales analysis report is published and subscribe an email address to it. The topic is responsible for delivering any message it receives to all of its subscribers. You use the Amazon SNS console for this task.

On the AWS Management Console, choose Services > Application Integration > Simple Notification Service.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/21583536-68c3-407d-a321-12546b117ce5)

In the navigation pane, choose Topics, and then choose Create topic.

Note: If the Topics link is not visible, choose the three horizontal lines icon, and then choose Topics.

The Create topic page opens.

Configure the following options:

Type: Choose Standard.

Name: Enter salesAnalysisReportTopic

Display name: Enter SARTopic
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8377df74-b8be-467c-b54e-9c692115ad79)

Choose Create topic.

Copy and paste the ARN value into a text editor document.
arn:aws:sns:us-west-2:767398040937:salesAnalysisReportTopic

You need to specify this ARN when you configure the next Lambda function

##  Subscribing to the SNS topic

    Choose Create subscription, and configure the following options:

Protocol: Choose Email.

Endpoint: Enter an email address that you can access.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7563c8b7-02b5-464e-ac88-496ef21b486f)

Choose Create subscription.

The subscription is created and has a Status of Pending confirmation.

Check the inbox for the email address that you provided.

You should see an email from SARTopic with the subject "AWS Notification - Subscription Confirmation."

Open the email, and choose Confirm subscription.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e402d454-7207-45d5-b1f4-8c4bebbab691)

A new browser tab opens and displays a page with the message "Subscription confirmed!"


# Creating the salesAnalysisReport Lambda function


you create and configure the salesAnalysisReport Lambda function. This function is the main driver of the sales analysis report flow. It does the following:

Retrieves the database connection information from Parameter Store

Invokes the salesAnalysisReportDataExtractor Lambda function, which retrieves the report data from the database

Formats and publishes a message containing the report data to the SNS topic


## Connecting to the CLI Host instance


you use EC2 Instance Connect to log in to the CLI Host instance running in your AWS account that already has the AWS Command Line Interface (AWS CLI) installed and the the Python code needed to create the next Lambda function. You then then run an AWS CLI command to create the Lambda function. Finally, you unit test the new function using the Lambda management console.


On the EC2 Management Console, in the navigation pane, choose Instances.

From the list of EC2 instances, choose the  check box for the CLI Host instance. 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7f8be858-d388-4d76-b2a8-91e364be4feb)

Choose Connect.

On the EC2 Instance Connect tab, choose Connect to connect to the CLI Host.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9d24aa38-4504-4425-968e-2308a81fbbdd)

# Configuring the AWS CLI

Amazon Linux instances have the AWS CLI pre-installed; however, you still need to supply credentials to connect the AWS CLI client to an AWS account.

In the EC2 Instance Connect terminal window, run the following command to update the AWS CLI software with the credentials:

    aws configure


    At the prompts, enter the following information:

AWS Access Key ID: At the top of these instructions, choose the Details dropdown menu, and then choose Show. A Credentials window opens. From this window, copy the AccessKey value, paste it into the terminal window, and press Enter.

AWS Secret Access Key: From the same Credentials window, copy the SecretKey value, paste it into the terminal window, and press Enter.
SecretKey	d3fSXNuYV9amz6vOLpI8WWwUMTHtgAN1XD/09ugO
CafePublicIP	34.221.112.173
LabRegion	us-west-2
CliHostPublicIP	54.212.110.116
AccessKey	AKIA3FLD42VURPSS2PHV
CafeServerPubDNS	ec2-34-221-112-173.us-west-2.compute.amazonaws.com

Default region name: Enter the Region code for the Region where you created the previous Lambda function. For this lab, enter the us-west-2 Region code into the terminal window, and press Enter. To find this code, in the upper-right menu of the Lambda management console, choose the Region dropdown menu. 

Default output format: Enter json and press Enter.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a6fbf30a-f1d7-4c1a-b720-c3532f37b371)
# Creating the salesAnalysisReport Lambda function using the AWS CLI
To verify that the salesAnalysisReport-v2.zip file containing the code for the salesAnalysisReport Lambda function is already on the CLI Host, run the following commands in the terminal:

       cd activity-files
         ls
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/da4c1d83-e53c-48e7-871c-2c8c2cc43e90)

Note: Before you create the function, you must retrieve the ARN of the salesAnalysisReportRole IAM role. You specify it in the following steps.

To find the ARN of an IAM role, open the IAM management console, and choose Roles. 


         ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4966ad44-b136-45a0-b2fc-73eeba6f6332)

In the search box, enter salesAnalysisReportRole, and choose the role name. The Summary page includes the ARN.    arn:aws:iam::767398040937:role/salesAnalysisReportRole


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/15723150-b16d-4619-b016-a661836293b4)


Next, you use the Lambda create-function command to create the Lambda function and configure it to use the salesAnalysisReportRole IAM role.


To do this, at the terminal window command prompt, paste the following command. Replace <salesAnalysisReportRoleARN> with the value of the salesAnalysisReportRole ARN that you copied in a previous step, and replace <region> with the us-west-2 Region code. This is the Region where you created the previous Lambda function. To find this code, in the upper-right menu of the Lambda management console, choose the Region dropdown menu.


aws lambda create-function \
--function-name salesAnalysisReport \
--runtime python3.9 \
--zip-file fileb://salesAnalysisReport-v2.zip \
--handler salesAnalysisReport.lambda_handler \
--region us-west-2 \
--role arn:aws:iam::767398040937:role/salesAnalysisReportRole



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ecbe14a4-683c-4a64-adb5-6d7dafb8d0e5)

Once the command completes, it returns a JSON object describing the attributes of the function. You now complete its configuration and unit test it.

## Configuring the salesAnalysisReport Lambda function

Open the Lambda management console.

Choose Functions, and then choose salesAnalysisReport. 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7dc87444-8ff9-4fac-b7e2-01402f3399c8)

The Details page for the function opens.

Review the details in the Function overview and Code source panels for the created function.

In particular, read through the function code, and use the embedded comments to help you understand the logic.

Notice on line 26 that the function retrieves the ARN of the topic to publish to, from an environment variable named topicARN. Therefore, you need to define that variable in the Environment variables panel.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9a0b2557-e1cd-4c83-b667-6313e7b0a3c0)

Choose the Configuration tab, and choose Environment variables.

Choose Edit.

Choose Add environment variable, and configure the following options:

Key: Enter topicARN

Value: Paste the ARN value of the salesAnalysisReportTopic SNS topic that you copied earlier.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/56ea65d1-cb4f-429b-b3e1-184d615ca1de)

Choose Save.

The following message appears: "Successfully updated the function salesAnalysisReport."
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/06821907-50a0-4946-a70f-445e4724a7c9)

## Testing the salesAnalysisReport Lambda function

Choose the Test tab, and configure the test event as follows:

For Test event action, choose Create new event.

For Event name, enter SARTestEvent

For Template, choose hello-world.

The function does not require any input parameters. Leave the default JSON lines as is.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/36168d80-5814-4d48-b933-e95b523091d5)

Choose Save.

Choose Test.

A green box with the message “Execution result: succeeded (logs)” appears.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/796af94c-268a-4bb4-b36f-0226cc9ff54c)

Tip: If you get a timeout error, choose the Test button again. Sometimes, when you first run a function, it takes a little longer to initialize, and the Lambda default timeout value (3 seconds) is exceeded. Usually, you can run the function again, and the error will go away. Alternatively, you can increase the timeout value. To do so, follow these steps:

Choose the Configuration tab.

Choose General configuration.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2fda1fed-839f-4f47-b8a0-3354ee64408a)

Choose Edit.

Adjust the Timeout as needed. 

Choose Save.
Choose Details to expand it.

The function should have returned the following JSON object:

       {
         "statusCode": 200,
         "body": "\"Sale Analysis Report sent.\""
        }
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7eee509a-7701-493c-8e9d-72e3ee30533e)


Check your email inbox.

If there were no errors, you should receive an email from AWS Notifications with the subject "Daily Sales Analysis Report."

The email should contain a report that is similar to the following image depending on the orders that you placed on the café website:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/bcf18aae-14e0-47aa-8a10-bea3ba5672d8)
You can place more orders on the café website and test the function to see the changes in the report that you receive.

Great job! You have successfully unit tested the salesAnalysisReport Lambda function.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/69b47b7a-b4b1-42a7-9ce2-f2dea98f971a)


## Adding a trigger to the salesAnalysisReport Lambda function
To complete the implementation of the salesAnalysisReport function, configure the report to be initiated Monday through Saturday at 8 PM each day. To do so, use a CloudWatch Events event as the trigger mechanism.
In the Function overview panel, choose Add trigger. The Add trigger panel is displayed.

In the Add trigger panel, configure the following options:

In the Trigger configuration pane, from the dropdown list, choose EventBridge (CloudWatch Events).

For Rule, choose Create a new rule. 

For Rule name, enter salesAnalysisReportDailyTrigger

For Rule description, enter Initiates report generation on a daily basis

For Rule type, choose Schedule expression.

For Schedule expression, specify the schedule that you would like by using a Cron expression. The general syntax of a Cron expression requires six fields separated by spaces as follows: 

cron(Minutes Hours Day-of-month Month Day-of-week Year): In addition, all times in a Cron expression are based on the UTC time zone.

Note: For testing purposes, enter an expression that schedules the trigger 5 minutes from the current time. You can use the following examples:

If you are in London (UTC time zone), and the current time is 11:30 AM, enter the following expression:

cron(35 11 ? * MON-SAT *)

If you are in New York (UTC time zone -5 during Eastern Standard Time), and the current time is 11:30 AM, enter the following expression:

cron(35 16 ? * MON-SAT *)

This Cron expression schedules the event to be invoked at 11:35 AM Monday through Saturday.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a0d14685-c3f8-4e61-a7bc-aaef84c8ac72)

Tip: For more information about the syntax of schedule expressions for rules, see Schedule Expressions for Rules.

To get the correct UTC time, navigate to any browser and enter UTC time

Choose Add.

The new trigger is created and displayed in the Function overview panel and Triggers pane.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/dbb842df-fbe9-4df5-9851-99b0da0471a9)

