# Challenge Lab: Amazon S3
I create an Amazon Simple Storage Service (Amazon S3) bucket and perform some routine tasks, such as uploading objects and configuring permissions to make those objects publicly accessible through a browser.


## Task 1: Connecting to the CLI Host instance

To start the challenge, I connect to the CLI Host instance that is already provisioned for me.

* **Step 1:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

* **Step 2:** In the navigation pane, choose **Instances**.

* **Step 3:** From the list of instances, select the **CLI Host instance**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/57fb5922-39da-4837-81fe-590c3d22c071)

* **Step 4:** Choose **Connect**.

* **Step 5:** On the **EC2 Instance Connect** tab, choose **Connect**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/11166a4c-0179-4224-9401-07f7345c983d)

Now that I am connected to the CLI Host instance, I can configure and use the AWS CLI to call AWS services.

## Task 2: Configuring the AWS CLI

* **Step 6:** To set up the AWS CLI profile with credentials, run the following command in the EC2 Instance Connect terminal:

          aws configure

 * **Step 7:** At the prompts, copy the following values that I pasted into my text editor, and paste them into the terminal window as directed.

   * **AWS Access Key ID**: Enter the value for **AccessKey:** 

   * **AWS Secret Access Key:** Enter the value for **SecretKey:** `gpV7KTRqGtZmrE9pzaCVWWsTgt0KvVbapHQ76iXl`.

   * **Default region name:** Enter `us-west-2`.

   * **Default output format:** Enter `json`.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ca793b5b-7cf7-453c-ac31-da88824b4386)

Now I am ready to run AWS CLI commands to interact with AWS services.


## Task 3: Finishing the challenge

To finish the challenge, do the following:

**Create an S3 bucket**.

  * **Step 8:** On the **AWS Management Console**, in the **Search** bar, enter and choose `S3` to open the **S3 Management Console**.
    
* **Step 9:** Create **bucket**
  
  * Bucketname: enter unique name
      
   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1e025360-4864-4c6a-9bba-9202ae7c104b)

   * Object Ownership: ACLs **enabled**
     
   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/dc1e900a-50ca-49ee-bf7a-7c3254ca2c72)

   * unbox block all public access
   * tick the acknowlege
     
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e62a475f-23c4-44cf-a34c-5d9f1f59983c)

    *  press **Create bucket**
      
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/65a828fa-ec87-48bc-8fc0-d18b75b7f1b9)

 **Upload an object into this bucket**.
 
* **Step 10:**  choose the bucket which I ealier created

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a767bbc2-3bc5-4532-9806-5da14dd09957)


 * **Step 11:**  choose **Upload**.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c693a365-cc23-470c-817d-42dfc56b8157)

* **Step 12:** choose **add files**

I add one jpg file you can upload any picture you want and upload it.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d53f4977-8b62-4a9b-ba0b-871250454fa8)

one image is upload.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e7eb71fc-3539-4f2d-b407-5c3bb2d5456a)


**Try to access the object by using a web browser.** 

* **Step 13:** In left navigation panel click on **Buckets**
  
Select bucket which I created and click on bucket name and select the file which I uploaded now and click on **Action** and select **make public using ACL**

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5513a695-f2c6-4239-bcc9-f5b02b34a183)

* **Step 14:** Now click **make public**

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c1834085-11a9-49f7-a82c-64116f0c052f)

* **Step 15:** click on **copy URL** and paste in a new browser.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a8b44e86-a077-45e7-921b-b9868d43d495)

I sucessfully able to see in the web browser  that image which i uploaded in s3 bucket.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4f9495c9-a8b9-4f7e-8c15-e17f52ec4cec)


Make the object (not the bucket) publicly accessible.

Access the object by using a web browser. 

* **Step 16:** I upload one more file
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a4104bab-0fb4-4f35-a2b6-7bfd94d72987)

**List the contents of the S3 bucket by using the AWS CLI.** 

* **Step 17:** I run this command

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ef12ec2a-1131-4df2-9359-85779826a594)

Now I am able to see list the contents of the s3 bucket by using the aws CLI

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/88806469-d2fb-4ee2-a368-24ad6c8b61ff)


 
