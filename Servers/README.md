# Creating a Website on S3
## Overview
I practice using AWS Command Line Interface (AWS CLI) commands from an Amazon Elastic Compute Cloud (Amazon EC2) instance to:
 * Create an Amazon Simple Storage Service (Amazon S3) bucket.
 * Create a new AWS Identity and Access Management (IAM) user that has full access to the Amazon S3 service.
 * Upload files to Amazon S3 to host a simple website for the Café & Bakery.
 * Create a batch file that can be used to update the static website when you change any of the website files locally.

   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/76e4ff94-6ce6-4965-8c1e-ac51c77ffbdb)

   Clients will be able to access the website I have deployed to Amazon S3. The website URL is similar to this example: 
   http://.s3-website-us-west-2.amazonaws.com. I can create and access the bucket through the AWS Management Console or 
    the AWS CLI.

   # Use SSH to connect to an Amazon Linux EC2 instance
    I loged in to an existing EC2 instance.
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
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5c3b4479-fc9f-49c2-87a9-95bb2749bd49)

# Configure the AWS CLI

Unlike some other Linux distributions that are available through Amazon Web Services (AWS), Amazon Linux instances already have the AWS CLI pre-installed on them.
* **Step 9:** In the SSH session terminal window, run the configure command to update the AWS CLI software with credentials.

                           aws configure

 * **Step 10:** At the prompt, configure the following:
               * AWS Secret Access Key: Copy and paste the SecretKey value into the terminal window.
               * Default region name: Enter us-west-2
               * Default output format: Enter json

   # Create an S3 bucket using the AWS CLI

   The s3api command creates a new S3 bucket with the AWS credentials in this lab. By default, the S3 bucket is created in 
    the us-east-1 Region.
   When I create a new S3 bucket, the bucket must have a unique name, such as the combination of my first initial, last name, and three random numbers. For example, if a user's name is Terry Whitlock, a bucket name could be `twhitlock256`
