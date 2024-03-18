# Monitoring Infrastructure
## Overview

The ability to monitor my applications and infrastructure is critical for delivering reliable, consistent IT services.

Monitoring requirements range from collecting statistics for long-term analysis to quickly reacting to changes and outages. Monitoring can also support compliance reporting by continuously checking that infrastructure is meeting organizational standards.

This lab shows  how to use Amazon CloudWatch Metrics, Amazon CloudWatch Logs, Amazon CloudWatch Events, and AWS Config to monitor my applications and infrastructure.

After completing this lab, I will be able to:

Use the AWS Systems Manager Run Command to install the CloudWatch agent on Amazon Elastic Compute Cloud (Amazon EC2) instances

Monitor application logs using CloudWatch agent and CloudWatch Logs

Monitor system metrics using CloudWatch agent and CloudWatch Metrics

Create real time notifications using CloudWatch Events

Track infrastructure compliance using AWS Config


## Task 1: Installing the CloudWatch agent

I can use the CloudWatch agent to collect metrics from EC2 instances and on-premises servers, including the following:

System-level metrics from EC2 instances, such as CPU allocation, free disk space, and memory utilization. These metrics are collected from the machine itself and complement the standard CloudWatch metrics that CloudWatch collects.

System-level metrics from on-premises servers that enable the monitoring of hybrid environments and servers not managed by AWS.

System and application logs from both Linux and Windows servers.

Custom metrics from applications and services using the StatsD and collectd protocols.

In this task, I use Systems Manager to install the CloudWatch agent on an EC2 instance. I configure it to collect both application and system metrics.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d46cd619-8781-4789-a064-b31376593c18)


* **Step 1:** In the **AWS Management Console**, on the **Services**  menu, select **Systems Manager**.

* **Step 2:** In the left navigation pane, choose **Run Command**.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ceae4b5e-0090-4378-816b-71918ce8967f)

 
* **Step 3:** Choose **Run a Command**

* **Step 4:** Select the button next to  **AWS-ConfigureAWSPackage** (typically appears toward the top of the list).  

* **Step 5:** Scroll to the **Command parameters** section and configure the following information:

    * Action: Select **Install**.

    * Name: Enter AmazonCloudWatchAgent

    * Version: Enter latest
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/43fad407-ac1d-44a6-a3ef-d54e674e5f3c)

* **Step 6:** In the **Targets** section, select **Choose instances manually**, and then under **Instances**, select the check box next to **Web Server**.

This configuration installs the CloudWatch agent on the web server.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3a8b6c2c-75d4-4ae1-bb77-88690910cf00)

* **Step 7:** At the bottom of the page, choose **Run**

* **Step 8:**  Wait for the **Overall status** to change to Success. You can occasionally choose  refresh toward the top of the page to update the status.

You can view the output from the job to confirm that it ran successfully.


* **Step 9:** Under Targets and outputs, choose  next to the instance, and then click View output.

* ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cb09b8ef-1d71-4b87-b6d7-3952a71f66d4)
  
* **Step 10:** Expand  Step 1 - Output.

 You should see the message Successfully installed arn:aws:ssm:::package/AmazonCloudWatchAgent.

 If you see the message Step execution skipped due to unsatisfied preconditions: '"StringEquals": [platformType, Windows]'. Step name: createDownloadFolder, then expand  Step 2 - Output instead. You can select this option because the instance you are using was created from a Linux AMI. You can safely ignore this message.

 You now configure the CloudWatch agent to collect the desired log information. The instance has a web server installed, so you configure the CloudWatch agent to collect the web server logs and general system metrics.

 You will store the configuration file in AWS Systems Manager Parameter Store, which the CloudWatch agent can then retrieve.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7f6b1cba-25e9-4828-9021-c26a33330201)


* **Step 11:** In the left navigation pane, choose Parameter Store.

* **Step 12:** Choose Create parameter, and then configure the following information:

Name: Enter Monitor-Web-Server

Description: Enter Collect web logs and system metrics

Value: Copy and paste the following configuration:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5fd67818-f640-4d4e-8384-f5c70a5e09c4)


20. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/524bf75a-4230-45f0-bccf-62e62bb9c70c)

24![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/95d8a09b-0611-4b8d-b3bf-30abfa31ad7e)

28![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/93b0f3d6-1bee-46e6-a1fd-004e0ec2fbae)


## Task 2: Monitoring application logs using CloudWatch Logs

You can use CloudWatch Logs to monitor applications and systems using log data. For example, CloudWatch Logs can track the number of errors that occur in your application logs and send you a notification whenever the rate of errors exceeds a threshold that you specify.

CloudWatch Logs uses your existing log data for monitoring, so no code changes are required. For example, you can monitor application logs for specific literal terms (such as "NullReferenceException") or count the number of occurrences of a literal term at a particular position in log data (such as 404 status codes in a web server access log). When the term you are searching for is found, CloudWatch Logs reports the data to a CloudWatch metric that you specify. Log data is encrypted while in transit and while it is at rest.

In this task, you generate log data on the Web Server and then monitor the logs using CloudWatch Logs.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/501668ef-9c4d-4f12-9f71-39aac6045aed)

The Web Server generates two types of log data:

Access logs

Error logs

You begin by accessing the web server.

Choose the Details dropdown menu above these instructions, and then choose Show

Copy the WebServerIP value.


WebServerIP	54.184.123.131
Open a new web browser tab, paste the WebServerIP you copied, and then press Enter.

You should see a web server Test Page.
You now generate log data by attempting to access a page that does not exist.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/79c77049-461d-4434-bed1-750f6a670480)


31. Append /start to the browser URL, and press Enter.

You receive an error message because the page is not found. This is okay! It generates data in the access logs that are being sent to CloudWatch Logs.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f8c16716-f1ba-4fab-b9e2-f94e920e32ea)

32.Keep this tab open in your web browser, but return to the browser tab showing the AWS Management Console.

33.From the Services  menu, choose CloudWatch.

34.In the left navigation pane, choose Log groups.

You should see two logs listed: HttpAccessLog and HttpErrorLog.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8f54db6d-7276-408e-b710-5f4ac77885aa)

 If these logs are not listed, try waiting a minute, and then choose  Refresh.


35.Choose HttpAccessLog (choose the name itself).

36. In the Logs streams section, choose the Log stream in the table (choose the name itself). It has the same ID as the EC2 instance that the log is attached to.

Log data should be displayed, consisting of GET requests that were sent to the web server. You can view additional information by choosing  to expand the lines. The log data includes information about the computer and the browser that made the request.

You should see a line with your /start request with a code of 404, which means that the page was not found.

This demonstrates how log files can be automatically shipped from an EC2 instance or an on-premises server to CloudWatch Logs. The log data is accessible without having to log in to each individual server. Log data can also be collected from multiple servers, such as an Auto Scaling fleet of web servers.


## Create a metric filter in CloudWatch Logs

i configure a filter to identify 404 Errors in the log file. This error would normally indicate that the web server is generating invalid links that users are choosing.

In the left navigation pane, choose Log groups.


Select the check box next to HttpAccessLog. 

From the Actions dropdown menu, select Create metric filter.

A filter pattern defines the fields in the log file and filters the data for specific values.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fb7e617b-2c4e-463c-acda-a60f75723b85)


Paste the following line into the Filter pattern box:

[ip, id, user, timestamp, request, status_code=404, size]
This line tells CloudWatch Logs how to interpret the fields in the log data and defines a filter to find lines only with status_code=404, which indicates that a page was not found.

In the Test pattern section, use the dropdown menu to select the EC2 instance id. It is be similar to i-0f07ab62aae4xxxx9.

Choose Test pattern

In the Results section, choose Show test results.

You should see at least one result with a $status_code of 404. This status code indicates that a page was requested that was not found.

44. Choose Next



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5ed58e02-6141-48ac-bfa7-074f574790de)

In the Create filter name section, in the Filter name box, enter 404Errors

In the Metric details section, configure the following information:

Metric namespace: Enter LogMetrics

Metric name: Enter 404Errors

Metric value: Enter 1

47. Choose Next. If Next is not enabled, click an empty text field, this will shift focus and enable it.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c10f6470-a7f0-44cb-abeb-4892529e2d6d)

48. On the Review and create page, choose Create metric filter

This metric filter can now be used in an alarm.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8f09ec5c-89ee-4b90-9d8f-712eaf3d54f5)

## Create an alarm using the filter

You now configure an alarm to send a notification when too many 404 Not Found errors are received.

49.In the 404Errors panel, choose the check box in the top-right corner.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9d1f6c15-c1e4-44fc-ba18-d506d2b902df)

50.In the Metric filters section, choose Create alarm

Configure the following settings:

In the Metrics section, for Period, select 1 minute.

In the Conditions section, select the following:

Whenever 404Errors is: Select  Greater/Equal

than: Enter 5

Choose Next

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ee4bd8eb-34ad-4fda-97ae-f5425c991c96)

52.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/eef0a8c1-7b56-40e7-99fc-b26ea8a1e6e5)

53.For Name and description, configure the following settings:

Alarm name: Enter 404 Errors

Alarm description: Enter Alert when too many 404s detected on an instance

Choose Next
Choose Create alarm

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e44658b9-0b22-4a8b-bdbe-0be4aa61bfdc)


Go to your email, look for a confirmation message, and select the Confirm subscription link.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/07e48b9b-c6f8-4272-82e9-550b4e418712)

56.Return to the AWS Management Console.


61.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/15e670cd-4ef2-478a-a878-2e7dcb477ea5)


Task 3: Monitoring instance metrics using CloudWatch

Metrics are data about the performance of your systems. CloudWatch stores metrics for the AWS services you use. You can also publish your own application metrics either via the CloudWatch agent or directly from your application. CloudWatch can present the metrics for search, graphs, dashboards, and alarms.

In the task, you use metrics that CloudWatch provides.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6909f266-13e9-41c4-a8a2-7bc10924382a)



65.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4c3249e8-5711-4041-a6be-2f81f1404ba1)


71.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5e97a0a1-c415-4cd3-abd4-3be1b3586912)


# Creating real time notifications

CloudWatch Events deliver a near-real-time stream of system events that describe changes in AWS resources. Simple rules can match events and route them to one or more target functions or streams. CloudWatch Events become aware of operational changes as they occur.

CloudWatch Events respond to these operational changes and take corrective action as necessary by sending messages to respond to the environment, activating functions, making changes, and capturing state information. You can also use CloudWatch Events to schedule automated actions that self trigger at certain times using cron or rate expressions.

In this task, you create a real time notification that informs you when an instance is stopped or terminated.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2b11efc3-c351-4b7e-87f9-0e47a5c23f98)


72. In the left navigation pane expand  Events, choose Rules.

# Configure a real time notification


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/70874a21-ddfa-4337-ba5e-e745ed386976)


# Monitoring for infrastructure compliance

















