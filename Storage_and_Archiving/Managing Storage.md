# Managing Storage

## overview

AWS provides multiple ways to manage data on Amazon Elastic Block Store (Amazon EBS) volumes. In this lab, I use AWS Command Line Interface (AWS CLI) to create snapshots of an EBS volume and configure a scheduler to run Python scripts to delete older snapshots.


In the challenge section of this lab, I challenged to sync the contents of a directory on an EBS volume to an Amazon Simple Storage Service (Amazon S3) bucket using an Amazon S3 sync command.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/44b72c7b-2308-4ffa-9c5f-c9c274450270)


My lab environment consists of a virtual private cloud (VPC) with a public subnet. Amazon Elastic Compute Cloud (Amazon EC2) instances named "Command Host" and "Processor" have already been created in this VPC for you as part of this lab.

The "Command Host" instance will be used to administer AWS resources including the "Processor" instance.


# Creating and configuring resources

I create an Amazon S3 bucket and configure the "Command Host" EC2 instance to have secure access to other AWS resources.


## Create an S3 bucket

 I create an S3 bucket to sync files from an EBS volume.

 * **Step 1:** On the **AWS Management Console**, in the **Search** bar, enter and choose `S3` to open the **S3 Management Console**.

* **Step 2:** On the console, choose **Create bucket**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4ba68872-b651-4a06-a7bb-52ddc7cd031d)

* **Step 3:** In the **Create bucket** section, configure the following:

  * **Bucket name**: Enter a bucket name. Use a combination of characters and numbers to keep it unique. 

 This will be referred to as "s3-bucket-name" throughout the lab.

   * **Region**: Leave as default.
   
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5e8dc20a-d0b2-43cb-916c-5701e00047e1)

* **Step 4:** Scroll and choose **Create bucket**.


## Attach instance profile to Processor


I attach a pre-created IAM role as an instance profile to the EC2 instance "Processor," giving it the permissions to interact with other AWS services such as EBS volumes and S3 buckets.

* **Step 5:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

* **Step 6:** In the navigation pane, choose **Instances**.

* **Step 7:** Choose **Processor** from the list of EC2 instances.

* **Step 8:** Choose **Actions > Security > Modify IAM role**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1000446b-7488-4a74-8fe8-dc0f7239282a)

* **Step 9:** Choose the `S3BucketAccess` role in the **IAM role** dropdown list.

* **Step 10:** Choose **Update IAM role**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f49d7415-d859-4657-b06b-f4e3572d3008)


# Taking snapshots of your instance

I use the AWS Command Line Interface (AWS CLI) to manage the processing of snapshots of an instance.


## Connecting to the Command Host EC2 instance

I use EC2 Instance Connect to connect to the "Command Host" EC2 instance. 

* **Step 11:** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

* **Step 12:** In the navigation pane, choose **Instances**.

* **Step 13:** From the list of instances, choose **Command Host**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b76c04ac-7dbb-4663-895e-da8871bdaf20)

* **Step 14:** Choose **Connect**.

* **Step 15:** On the **EC2 Instance Connect** tab, choose **Connect**.

This option opens a new browser tab with the **EC2 Instance Connect** terminal window.

**Note:** If I prefer to use an SSH client to connect to the EC2 instance, see the guidance provided in the additional references section.

I use this terminal window to complete the tasks throughout the lab. If the terminal becomes unresponsive, refresh the browser or use the steps in this task to connect again.


## Taking an initial snapshot

I identify the EBS volume that's attached to the "Processor" instance and take an initial snapshot. To do so, I run commands in the EC2 Instance Connect terminal window. I can copy the command output to a text editor for subsequent use.


* **Step 16:** To display the EBS volume-id, run the following command: 


       aws ec2 describe-instances --filter 'Name=tag:Name,Values=Processor' --query 
       'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.{VolumeId:VolumeId}'



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8124443e-cff1-4a0a-b80f-6e7baefe1298)


**Note:** The command returns a response : "VolumeId": **"vol-0772600274d66bec8"**. 

I use this value for VolumeId throughout the lab steps when prompted.



* **Step 17:**  I take snapshot of this volume. Prior to this, I shut down the "Processor" instance, which requires its instance ID.  Run the following command to obtain the instance ID:

       aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' -- 
       query  'Reservations[0].Instances[0].InstanceId'


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5810690c-cd3e-49ed-96b4-d190af9cc228)

 The command returns a value for INSTANCE-ID : **"i-0179ed69870b42fc1"**

* **Step 18:** To shut down the "Processor" instance, run the following command and replace "INSTANCE-ID" with the instance-id that I retrieved earlier:

       aws ec2 stop-instances --instance-ids INSTANCE-ID

INSTANCE-ID : **"i-0179ed69870b42fc1"**

      aws ec2 stop-instances --instance-ids i-0179ed69870b42fc1

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/416e899e-7e5d-491e-9cad-0f2654e53821)



* **Step 19:**  To verify that the "Processor" instance stopped, run the following command and replace "INSTANCE-ID" with your instance id.


       aws ec2 wait instance-stopped --instance-id INSTANCE-ID

INSTANCE-ID : **"i-0179ed69870b42fc1"**

    aws ec2 wait instance-stopped --instance-id i-0179ed69870b42fc1

When the instance stops, the command returns to a prompt.

* **Step 20:** To create my first snapshot of the volume of your "Processor" instance, run the following command and replace "VOLUME-ID" with the VolumeId that you retrieved earlier:

       aws ec2 create-snapshot --volume-id VOLUME-ID
  
"VolumeId": **"vol-0772600274d66bec8"**

      aws ec2 create-snapshot --volume-id vol-0772600274d66bec8

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0619fc30-e6b8-40f1-ad8e-3a686ebbf510)



* **Step 21:** To check the status of my snapshot, run the following command and replace "SNAPSHOT-ID" with the SnapshotId that I retrieved earlier:

       aws ec2 wait snapshot-completed --snapshot-id SNAPSHOT-ID
  
snapshot-id:**snap-0ff15a37ee2295280**

        aws ec2 wait snapshot-completed --snapshot-id snap-0ff15a37ee2295280
  
* **Step 22:** To restart the "Processor" instance, run the following command and replace "INSTANCE-ID" with the instance-id that I retrieved earlier:

       aws ec2 start-instances --instance-ids INSTANCE-ID
  
INSTANCE-ID : **"i-0179ed69870b42fc1"**

       aws ec2 start-instances --instance-ids i-0179ed69870b42fc1
      
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5eec8002-5f39-42f8-b806-70ef3406eed2)



## Scheduling the creation of subsequent snapshots


Using the Linux scheduling system (cron), I can set up a recurring snapshot process so that new snapshots of my data are taken automatically.

For the purposes of this lab, I schedule a snapshot creation every minute so that I can verify the results of my work.

In this task, I create a cron job to manage the number of snapshots that are maintained for a volume.

**Note:** This section of the lab doesn't stop the instance in order to create a large number of snapshots for the next step.


* **Step 23:** To create and schedule a cron entry that runs a job every minute, run the following command and replace "VOLUME-ID" with the VolumeId that you retrieved earlier:
  
      echo "* * * * *  aws ec2 create-snapshot --volume-id VOLUME-ID 2>&1 >> 
      /tmp/cronlog" > cronjob
      crontab cronjob

 "VolumeId": **"vol-0772600274d66bec8"**


       echo "* * * * *  aws ec2 create-snapshot --volume-id vol-0772600274d66bec8 
      2>&1 >> /tmp/cronlog" > cronjob
       crontab cronjob


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/64491388-55e0-44bd-88e2-dbc1dc9ab46a)

* **Step 24:** To verify that subsequent snapshots are being created, run the following command and replace "VOLUME-ID" with the VolumeId that you retrieved earlier:

       aws ec2 describe-snapshots --filters "Name=volume-id,Values=VOLUME-ID"

  "VolumeId": **"vol-0772600274d66bec8"**
  
         aws ec2 describe-snapshots --filters "Name=volume-id,Values=vol-0772600274d66bec8"
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/bdcb3d0c-32e8-4a71-bcec-995e7916cbaf)

Re-run the command after few minutes to see more snapshots.

Wait a few minutes so that a few more snapshots are generated before beginning the next task.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d8118301-0255-4f72-be5a-ea5b96ab8c34)


### Retaining the last two snapshots
I run a Python script that maintains only the last two snapshots for any given EBS volume.
* **Step 25:** To stop the cron job, run the following command:

         crontab -r

* **Step 26:** To examine the contents of the Python script "snapshotter_v2.py", run the following command:

          more /home/ec2-user/snapshotter_v2.py

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c3803207-a185-4c54-bd39-38a1f95bf143)

* **Step 27:** Before running snapshotter_v2.py, run the following command and replace "VOLUME-ID" with the VolumeId that you retrieved earlier:
  
       aws ec2 describe-snapshots --filters "Name=volume-id, Values=VOLUME-ID" -- 
       query 'Snapshots[*].SnapshotId'
  
"VolumeId": **"vol-0772600274d66bec8"**

        aws ec2 describe-snapshots --filters "Name=volume-id, Values=vol- 
         0772600274d66bec8" --query 'Snapshots[*].SnapshotId'

The command returns the multiple snapshot IDs that were returned for the volume. These are the snapshots that were created by your cron job before you stopped it.

* **Step 28:** Run the the "snapshotter_v2.py" script using following command:


       python3 snapshotter_v2.py
  
![ image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8356dd57-badb-499b-a367-3e2b8c8f8071)

* **Step 29:** To examine the new number of snapshots for the current volume, re-run the following command from an earlier step:


       aws ec2 describe-snapshots --filters "Name=volume-id, Values=VOLUME-ID" -- 
       query 'Snapshots[*].SnapshotId'

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/61ff7e08-fcd3-4d8c-9b76-2801810ffb00)


### Challenge: Synchronize files with Amazon S3
I challenged to sync the contents of a directory with the Amazon S3 bucket that I created earlier.

## Challenge Description

Run the following command in the terminal to download a sample set of files:

      wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-
      1-23732/183-lab-JAWS-managing-storage/s3/files.zip
      
Unzip these files, and then, using the AWS CLI, figure out how to accomplish the following:

  * Activate versioning for your Amazon S3 bucket.

  * Use a single AWS CLI command to sync the contents of your unzipped folder with 
   your Amazon S3 bucket.

  * Modify the command so that it deletes a file from Amazon S3 when the 
    corresponding file is deleted locally on your instance.

  * Recover the deleted file from Amazon S3 using versioning.

I can use the solution summary as a guide to complete the challenge myself. Use the links in additional references section for details on using required AWS CLI commands.

### Solution Summary
The solution involves the following steps:
* To activate versioning for the bucket, use the aws s3api put-bucket-versioning command.

* To synchronize the local files with Amazon S3, use the aws s3 sync command on the local folder.

* Delete a local file.

* To force Amazon S3 to delete any files that aren't present on the local drive but present in Amazon S3, use the --delete option with the aws s3 sync command.	

* Because there's no direct command in Amazon S3 to restore a previous version of a file, to download a previous version of the deleted file from Amazon S3, use the aws s3api list-object-versions and aws s3api get-object commands. You can then restore the file to Amazon S3 by using  aws s3 sync.

  ## Downloading and unzipping sample files

  The sample file package contains a folder with three text files: file1.txt, file2.txt, and file3.txt. These are the files that I will sync with my Amazon S3 bucket.


 * **Step 30:** Connect to the "Processor" instance using EC2 Instance Connect.

**Note**: Refer to the earlier steps that I used to connect to the "Command Host" instance.

I run following AWS CLI commands in the EC2 Instance Connect terminal window.

* **Step 31:** To download the sample files on the "Processor" instance, run the following command from within your instance:

      wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-
      1-23732/183-lab-JAWS-managing-storage/s3/files.zip    
    

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0ea2e6fe-7db2-4f37-b9bc-03ed70eeec94)

* **Step 32:** To unzip the directory, use the following command:
             unzip files.zip
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/434d3e9c-b130-4402-8122-5aec100dd603)

## Syncing files

* **Step 33:** Before syncing content with my Amazon S3 bucket, I need to activate versioning on your bucket. 

Run the following command and replace "S3-BUCKET-NAME" with my bucket name:

    aws s3api put-bucket-versioning --bucket S3-BUCKET-NAME --versioning- 
    configuration Status=Enabled
    
S3-BUCKET-NAME: **s3-bucket-name-07**

     aws s3api put-bucket-versioning --bucket s3-bucket-name-07 --versioning- 
     configuration Status=Enabled

* **Step 34:** To sync the contents of the files folder with my Amazon S3 bucket, run the following command and replace "S3-BUCKET-NAME" with my bucket name:

        aws s3 sync files s3://S3-BUCKET-NAME/files/
    
S3-BUCKET-NAME: **s3-bucket-name-07**

         aws s3 sync files s3://s3-bucket-name-07/files/
         
The command confirms that three files were uploaded to my S3 bucket.

To confirm the state of my files, run the following command and replace "S3-BUCKET-NAME" with my bucket name:
         
* **Step 14**
39. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3f19a64f-647a-4c47-afc9-de1bbce9bc67)
* **Step 14**
40. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c072fc84-5d63-4b56-8f2d-d4a50c59c7b1)
* **Step 14**
42. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cc2d36fa-81a8-4aec-9e00-977cf73febb5)
* **Step 14**
44. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b98a8a7f-47dd-4b3e-b6b2-7a411b8f8360)


* **Step 14**
45.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0791cd62-e52b-4960-83f1-868197e433ce)
* **Step 14**
46.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/143365ca-e16f-4419-8175-d1d90dfa3668)

* **Step 14**
47.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f2f0193f-f3b0-442f-b535-854aeb50eff4)

* **Step 14**
48. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6d75bfa4-a321-4d5c-9c0e-aee889c4658a)








