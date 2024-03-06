# Amazon Route 53 Failover Routing

## overview

I configured failover routing for a simple web application.

The activity environment starts with two Amazon Elastic Compute Cloud (Amazon EC2) instances that have already been created. Each of the instances has the full LAMP stack installed and the café website deployed and running. The EC2 instances are deployed in different Availability Zones. For example, if the web servers are running in the us-west-2 Region, then one of the web servers runs in the us-west-2a Availability Zone and the other one runs in the us-west-2b Availability Zone.

I will configure my domain such that, if the website in the primary Availability Zone becomes unavailable, Amazon Route 53 will automatically fail over application traffic to the instance in the secondary Availability Zone.

When I are finished, your environment will look like the following architecture:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7fa39e4c-7af1-4ddb-88cd-acd8d3e11813)


The architecture diagram shows the final state of the infrastructure that I build. Route 53 records store the IP address of the EC2 instance in each Availability Zone. User requests are normally sent to the IP address corresponding to Café Instance1 in Availability Zone 1. If Café Instance1 is unavailable, requests are routed to Café Instance2 in Availability Zone 2 based on the configuration in the Route 53 records. When Café Instance1 becomes unavailable, a Route 53 health check alarm is invoked, and an email alert is sent to the email address provided.
# Confirming the café websites


 I analyze the resources that AWS CloudFormation has automatically created for me.

 * **Step 1:** At the top of this page, choose **Details**. For **AWS**, choose **Show**. A **Credentials** panel opens.

* **Step 2:** Copy the values for the following parameters, and paste them into a text editor to use later.

  * CafeInstance1IPAddress : 54.70.1.218

  * PrimaryWebSiteURL : 54.70.1.218/cafe

  * SecondaryWebsiteURL : 52.38.183.115/cafe

  * CafeInstance2IPAddress : 52.38.183.115

* **Step 3:** Choose **X** to close the **Credentials** panel.

* **Step 4:** Navigate to the browser tab with the **AWS Management Console**. In the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

* **Step 5:** In the left navigation pane, in the **Instances** section, choose **Instances**.

Two EC2 instances have already been created for me. **CafeInstance1** is running in Cafe Public Subnet 1 (us-west-2a), and **CafeInstance2** is running in Cafe Public Subnet 2 (us-west-2b).

The URLs that I copied earlier correspond to the café application running on each instance.

Although both EC2 instances have the same configuration and application installed, one instance is a primary instance.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/31870788-0679-4d51-8158-c041578fe6f1)


* **Step 6:** Open a new browser tab, and paste the value for **PrimaryWebSiteURL**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d52f735a-93ea-46d5-a509-aac3315df374)

   * The café application webpage should open.

   * Along with other information about the café, notice the Server Information that is displayed. It 
     shows information about the EC2 instance and the Availability Zone where it is running.



* **Step 7:** Open another browser tab, and paste the value for **SecondaryWebsiteURL**. Confirm that the second EC2 instance has similar configurations as the first instance.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/95ea38c5-a15b-4473-b02e-3f63babec0b0)

These configurations confirm that the café application is running on both instances.

* **Step 8:** On one of the websites, choose **Menu**.


* **Step 9:** Choose any item on the menu, and choose **Submit Order**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e7e1ac8a-1a0c-4430-b720-227f5b3ad6d3)

The **Order Confirmation** page reflects the time that the order was placed in the time zone where the web server is running.


You have now confirmed that two instances are running the café application. Each application is running in a different Availability Zone to provide high availability.


# Configuring a Route 53 health check

The first step to configure failover is to create a health check for your primary website.

* **Step 10:** In the AWS Management Console, from the **Services** menu, enter and choose `Route 53` to open the **Route 53 Management Console**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a5a500de-6f49-4ed8-8a1a-70c1e868f0c6)

* **Step 11:** In the left navigation pane, choose **Health checks**.

* **Step 12:** Choose **Create health check**, and configure the following options. Leave the default values for all other fields.

  * **Name:** Enter `Primary-Website-Health` 

  * **What to monitor:** Choose **Endpoint**.

  * **Specify endpoint by:** Choose **IP address**.

  * **IP address:** Paste in the **Public IPv4 address** of **CafeInstance1**. You can find this value in the EC2 console, or you can copy the IP address from the **CafeInstance1IPAddress** value that you copied earlier.

  * **Path:** Enter `cafe`

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/689d3725-850b-4d4c-8970-ed1a4c695970)

* **Step 13:** Expand  **Advanced configuration**, and configure the following options. Leave the default values for all other fields.

  * **Request interval:** Choose **Fast (10 seconds).**

  * **Failure threshold:** Enter `2`

These options make your health check respond faster.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2d0849d5-419e-4a9d-a4bd-d8ac4a7bcc30)

* **Step 14:** Choose **Next.**


* **Step 15:** For **Get notified when health check fails**, configure the following options:

  * **Create alarm:** Choose **Yes**.

  * **Send notification to:** Choose **New SNS topic**.

  * **Topic name:** Enter `Primary-Website-Health`

  * **Recipient email address:** Enter an email address that you can access.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/03bf4bf6-8c4c-4928-8b72-8e46fcd1444e)


* **Step 16:** Choose **Create health check**.

Route 53 now checks the health of your site by periodically requesting the domain name that you provided and verifying that it returns a successful response.

The health check might take up to a minute to show a Healthy **Status**. Choose the refresh icon to update your view of the current status.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/44f87ef6-049f-434b-b24b-f3d90ff0d9f1)



* **Step 17:** Select **Primary-Website-Health**, and then choose the **Monitoring** tab.

This tab provides a view of the status of the health check over time. It might take a few seconds before the chart becomes available. Choose the refresh icon to update my view.



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5841b6e1-b72a-41be-bed7-4a1bf0a8cb95)

* **Step 18:** Check my email. I should have received an email from AWS Notifications.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/44237d1e-db9d-411b-a45f-83b5fc83fbee)

* **Step 19:** In the email, choose the **Confirm subscription** link to finish setting up the email alerting that you configured when you created the health check.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0bb8c9ef-dd5d-41a6-8c91-7dad264e786f)



# Configuring Route 53 records

 I created Route 53 records for the hosted zone.

 ## Creating an A record for the primary website

 I configure failover routing based on the health check that I just created.

 * **Step 20:** In the Route 53 console, in the left navigation pane, choose **Hosted zones**.
   
 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8d4d9b77-9dc4-4176-bfb2-bc3d098762c4)


The domain name **XXXXXX_XXXXXXXXXX.vocareum.training** (where the Xs are digits unique to your AWS account) has already been created for you.

All lab participants have been given a unique domain name.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/81a4eff0-dc1f-4e7a-8ac4-16b04442e637)

* **Step 21:** Choose **XXXXXX_XXXXXXXXXX.vocareum.training** to display the two records that already exist in this hosted zone. 

These two records were created when the domain was registered with Route 53. The NS, or name server record, lists the four name servers that are the authoritative name servers for your hosted zone in the Value/Route traffic to column. You should not add, change, or delete name servers from this record.

The SOA, or start of authority record, identifies the base Domain Name System (DNS) information about the domain in the **Value/Route traffic to** column. It was also created when the domain was registered with Route 53.




* **Step 22:** Choose **Create record**, and configure the following options: 

  * **Record name:** Enter `www`

  * **Record type:** Choose A - Routes traffic to an IPv4 address and some AWS resources.

  * **Value:** In the text box, enter the IP address for CafeInstance1IPAddress.

  * **TTL (seconds):** Enter `15`

  * **Routing policy:** Choose **Failover**.

  * **Failover record type:** Choose Primary.

  * **Health check ID:** Choose **Primary-Website-Health**.

  * **Record ID:** Enter `FailoverPrimary`


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d9e6d4a8-fe50-4f02-af62-ef96d36d41fb)

* **Step 23:** Choose **Create records**.

The A-type record that you created should now appear as the third record on the **Hosted zones** page.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8167cd2b-37c0-414b-a68f-188deddda29e)

# Creating an A record for the secondary website

Now you create another record for the stand-by/secondary web server.

* **Step 24:** Choose **Create record**, and configure the following options: 

     * **Record name:** Enter www

     * **Record type:** Choose A - Routes traffic to an IPv4 address and some AWS resources.

     * **Value:** In the text box, enter the IP address for CafeInstance2IPAddress. To find this value, at the top of these instructions, choose Details, and then choose Show, or copy it from the values that you pasted into a text editor earlier in the lab.

* **TTL (seconds):** Enter 15

* **Routing policy:** Choose **Failover**.

* **Failover record type:** Choose **Secondary**.

* **Health check ID:** Leave this field empty.

* **Record ID:** Enter FailoverSecondary

* **Step 25:** Choose **Create records**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/978f3df2-c6df-46ab-9016-347e48b0e195)


Another A-type record should now be listed on the **Hosted zones** page.

Now configured my web application to fail over to another Availability Zone.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d3259e95-744e-464a-86fe-f10939f1b1eb)

# Verifying the DNS resolution

I visited the DNS records in a browser to verify that Route 53 is pointing correctly to your primary website.

* **Step 26:** Select the check box for either one of the A records. A **Record details** panel appears that includes the **Record name**. Copy the **Record name** value of the A record.

* **Step 27:** Open a new browser tab. Paste the A record name, enter `/cafe` at the end of the URL, and then load the page.

The café primary website should load, as indicated by the **Server Information** section of the page, which should display the **Region/Availability Zone**.

**Tip:** The URL should be http://www.XXXXXX_XXXXXXXXXX.vocareum.training/cafe/, and in this URL, the Xs are unique digits.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/eb67f563-52f4-4894-8b2f-ee9d1841e36e)


# Verifying the failover functionality

I try to verify that Route 53 correctly fails over to my secondary server if my primary server fails. For the purposes of this activity, I simulate a failure by manually stopping **CafeInstance1**.

Return to the AWS Management Console. On the **Services** menu, enter and choose EC2 and then choose **Instances**.

* **Step 27:** Select **CafeInstance1**.

* **Step 28:** From the **Instance state** menu, choose **Stop instance**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/112dd19b-9877-465a-b3fc-9a69242a2389)


* **Step 29:** In the **Stop instance?** window, choose **Stop**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cece56b5-ab35-4c22-bdbb-bc9dfda4ac59)


The primary website now stops functioning. The Route 53 health check that you configured notices that the application is not responding, and the record entries that you configured cause DNS traffic to fail over to the secondary EC2 instance.


* **Step 30:** On the **Services** menu, enter and choose `Route 53`

* **Step 31:** In the left navigation pane, choose **Health checks**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e66b6ea0-df93-49b8-ba6b-46cadcba3dcc)


* **Step 32:** Select  **Primary-Website-Health**, and in the lower pane, choose the Monitoring tab.

I should see failed health checks within minutes of stopping the EC2 instance.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/95158913-c92f-4a7d-9b09-2665bebafd32)

* **Step 33:** Wait until the **Status** of **Primary-Website-Health** is Unhealthy. If necessary, periodically choose  refresh. It might take a few minutes for the status to update.


* **Step 34:** Return to the browser tab where you have the **vocareum_XXXXXX_XXXXXXXXXX.training/cafe** website open, and refresh the page.

Notice that the **Region/Availability Zone** value now displays a different Availability Zone (for example, us-west-2b instead of us-west-2a). You are now seeing the website served from your **CafeInstance2** instance.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/28de4b01-12b3-43b0-8bb3-fd3a8dab0cf2)


If you do not get the correct results, reconfirm that the **Status** of **Primary-Website-Health** is Unhealthy, and then try again. It might take a few minutes for the DNS changes to propagate.

Check your email. You should have received an email from AWS Notifications titled "ALARM: Primary-Website-Health-awsroute53-..." with details about what initiated the alarm.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0806c3e6-32d2-4930-acad-e3e3acc898c9)


Note: It might take a few minutes before the email arrives.

I successfully confirmed that my application environment can fail over from its primary Availability Zone to its secondary Availability Zone if the server in the primary Availability Zone fails.

 














