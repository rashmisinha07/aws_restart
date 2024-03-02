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
   When I create a new S3 bucket, the bucket must have a unique name, such as the combination of my first initial, last 
    name, and three random numbers. For example, if a user's name is Terry Whitlock, a bucket name could be `twhitlock256`

   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3ebb7c80-6d2f-41cd-bd11-3534083ebc58)
   
  * **Step 11:** To create a bucket in Amazon S3, I use the aws s3api create-bucket command. When I use this command to 
                 create an S3 bucket, you also include the following:

    * Specify `--region us-west-2`
    * Add `--create-bucket-configuration LocationConstraint=us-west-2` to the end of the command.
      
    The following is an example of the command to create a new S3 bucket. I can use rashmi07 as my bucket name.

           aws s3api create-bucket --bucket rashmi07 --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2


If the command is successful, I will get a JSON-formatted response with a Location name-value pair, where the value reflects the bucket name. The following is an example:

    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/2a2eea04-adaf-4694-a232-46ca078f7fc0)

   # Create a new IAM user that has full access to Amazon S3
   The AWS CLI command: `aws iam create-user` creates a new IAM user for my AWS account. The option `--user-name` is used to create the name of the user and must be unique within the account. 

   * **Step 12:** Using the AWS CLI, create a new IAM user with the command `aws iam create-user` and username `awsS3user`:

          aws iam create-user --user-name awsS3user

     ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fc5b0c2a-4442-491c-8375-33a8c50dbe52)


   * **Step 13:**  Create a login profile for the new user by using the following command:

           aws iam create-login-profile --user-name awsS3user --password Training123!

     ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9a36a1dd-b9db-45e2-aec5-e3fe615cc9e7)


   * **Step 14:** Copy the AWS account number:
     *  In the AWS Management Console, choose the account voclabs/user... drop down menu located at the top right of the screen.
     *  Copy the 12 digit Account ID number.
     *  In the current drop down menu, choose Sign Out.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/dc9029f4-82a5-41c6-81d5-940f41532408)

* **Step 15:** Log in to the AWS Management Console as the new awsS3user user:
    * In the browser tab where you just signed out of the AWS Management Console, choose Log back in or Sign in to the Console.
    * In the sign-in screen, choose the radio button IAM user.
    * In the text field, paste or enter the account ID with no dashes.
    * Choose Next

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c97a21bb-37a9-4f50-ab03-4ffd03e998e0)

 * A new login screen with Sign in as IAM user field will show. The account ID will be filled in from the previous screen.
    * For **IAM user name**, enter `awsS3user`
    * For **Password**, enter `Training123!`
    * Choose **Sign In**

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3f17d611-97b3-4204-8a3a-161c684438af)

* **Step 16:** On the AWS Management Console, in the Search box, enter S3 and choose S3. This option takes me to the Amazon S3 console page.
  **Note**: The bucket that I created might not be visible. Refresh the Amazon S3 console page to see if it appears. The **awsS3user** user does not have Amazon S3 access to the bucket that you created, so you might see an error for **Access** to this bucket.

* **Step 17:** In the terminal window, to find the AWS managed policy that grants full access to Amazon S3, run the following command:
 
           aws iam list-policies --query "Policies[?contains(PolicyName,'S3')]"

    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/16b99e53-34a9-418f-9dad-856b25ef0d63)

    The result displays policies that have a PolicyName attribute containing the term S3. Locate the policy that grants full access to Amazon S3. I use this policy in the next step.

* **Step 18:**  To grant the awsS3user user full access to the S3 bucket, replace <policyYouFound> in following command with the appropriate PolicyName from the results, and run the adjusted command:
 
         
         aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --user-name awsS3user
 

* **Step 19:** Return to the AWS Management Console, and refresh the browser tab.

  If you implemented the correct policy, the **Access** portion of the bucket now has **Objects can be public**.
  
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8d296086-78d8-4f64-b14f-a2bc9607f45e)


  # Extract the files that you need for this lab

  A file containing the `static-website` contents for the Amazon S3 bucket will need to be extracted in the following step.

* **Step 20:** Back in the SSH terminal, extract the files that I need for this lab by running the following commands:

            cd ~/sysops-activity-files
            tar xvzf static-website-v2.tar.gz
            cd static-website


  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8a8b5077-3096-4646-b9d1-a33e14722a07)

* **Step 21:** To confirm that the files were extracted correctly, run the `ls` command. 

I should see a file named index.html and directories named css and images.

# Upload files to Amazon S3 by using the AWS CLI

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d5393df6-79bb-4bb8-a009-47c30315fbee)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/058b3c91-d7f0-4e4f-9fff-5a173ca81f0c)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7138a5a7-8f3d-4fd1-b236-49647c8ea208)



Once the files are extracted, I upload the contents of the file to Amazon S3. These files include what I explored when I ran the ls command.

* **Step 22:** So that the bucket can function as a website, replace <my-bucket> in the following command with your bucket name, and run the adjusted command.

            aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://rashmi07/ --recursive --acl public-read

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/346ab6fc-56ea-44e3-8acc-8017da326279)

Notice that the upload command includes an access control list (ACL) parameter. This parameter specifies that the uploaded files have public read access. It also includes the recursive parameter, which indicates that all files in the current directory on your machine should be uploaded.



* **Step 23:** To verify that the files were uploaded, replace <my-bucket> in the following command with my bucket name, and run the adjusted command:

                 aws s3 ls rashmi07

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8d796c78-83e1-42a8-960d-c2851d121187)


* **Step 24:** On the AWS Management Console, on the Amazon S3 console, choose my bucket name.
 
    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1b703718-2894-44ed-979d-6fe405bdbb27)

 * **Step 25:** Choose the Properties tab. At the bottom of the this tab, note that Static website hosting is Enabled. Running the aws s3 website AWS CLI command turns on the static website hosting for an Amazon S3 bucket. This option is usually turned off by default.

    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/64e4c978-3d68-436f-b99a-92133ca1eefa)

* **Step 26:** To open the URL on a new page, choose the Bucket website endpoint URL that displays.

    
    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/623d3b39-5eb4-435c-a9c1-624f903316b8)


   I have created a static website that is available to the public for viewing!

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ea84cdb2-733b-4419-92cd-174767245779)


  # Create a batch file to make updating the website repeatable

  To create a repeatable deployment, I created a batch file by using the VI editor.

* **Step 27:** In the terminal window, to pull up the history of recent commands, run the following command:
 
              history

    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/767391c9-ba89-43f8-ae77-988e6112cbb4)

* **Step 28:** Locate the line where I ran the aws s3 cp command. I will put this line in my new batch file.

 * **Step 29:** To change directories and create an empty file, run the following command in the SSH terminal session:

                 cd ~
                 touch update-website.sh


* **Step 30:**  To open the empty file in the VI editor, run the following command.

                vi update-website.sh


* **Step 31:** To enter edit mode in the VI editor, press i
       
* **Step 32:** To write the changes and quit the file, press Esc, enter :wq and then press Enter.
 
* **Step 33:**  Next, I add the standard first line of a bash file and then add the s3 cp line from my history. To do so, replace <my-bucket> in the following command with my bucket name, and copy and paste the adjusted command into my file:
 
            aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://rashmi07/ --recursive --acl public-read


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fc823ed7-d505-4ed9-afcc-5e62b5c354aa)


* **Step 34:** To make the file an executable batch file, run the following command:

         chmod +x update-website.sh

* **Step 35:** To open the local copy of the index.html file in a text editor, run the following command:
 
                    vi sysops-activity-files/static-website/index.html


    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5a107c6c-f14d-4689-aa7c-3fa0402aeb05)

* **Step 36:**  To enter edit mode in the VI editor, press i and modify the file as follows:

  * Locate the first line that has the HTML code bgcolor="aquamarine" and change this code to bgcolor="gainsboro"

  * Locate the line that has the HTML code bgcolor="orange" and change this code to bgcolor="cornsilk"

  * Locate the second line that has the HTML code bgcolor="aquamarine" and change this code to bgcolor="gainsboro"

  * To write the changes and quit the file, press Esc, enter :wq and then press Enter.
 
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/bacda38d-3d3d-445c-8d48-006080664bb6)

* **Step 37:** To update the website, run your batch file.

          ./update-website.sh

* **Step 38:** To see the changes to the website, return to the browser and refresh the Café and Bakery page.

I just made your first revision to the website!


  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8ab91d1c-b02e-4623-984e-5f8445bbbc59)


  # Conclusion

  I have successfully done the following:

   * Ran AWS CLI commands that use IAM and Amazon S3 services
   * Deployed a static website to an S3 bucket
   * Created a script that uses the AWS CLI to copy files in a local directory to Amazon S3




    














                     
