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

 * **Step 1** On the **AWS Management Console**, in the **Search** bar, enter and choose `S3` to open the **S3 Management Console**.

* **Step 2** On the console, choose **Create bucket**.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4ba68872-b651-4a06-a7bb-52ddc7cd031d)

* **Step 3** In the **Create bucket** section, configure the following:

 * **Bucket name**: Enter a bucket name. Use a combination of characters and numbers to keep it unique. 

 This will be referred to as "s3-bucket-name" throughout the lab.

 * **Region**: Leave as default.
   
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5e8dc20a-d0b2-43cb-916c-5701e00047e1)

* **Step 4** Scroll and choose **Create bucket**.


# Attach instance profile to Processor


I attach a pre-created IAM role as an instance profile to the EC2 instance "Processor," giving it the permissions to interact with other AWS services such as EBS volumes and S3 buckets.

* **Step 5** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

  **Step 6** In the navigation pane, choose **Instances**.

* **Step 7** Choose **Processor** from the list of EC2 instances.

* **Step 8** Choose **Actions > Security > Modify IAM role**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1000446b-7488-4a74-8fe8-dc0f7239282a)

* **Step 9** Choose the `S3BucketAccess` role in the **IAM role** dropdown list.

* **Step 10** Choose **Update IAM role**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f49d7415-d859-4657-b06b-f4e3572d3008)


## Taking snapshots of your instance

I use the AWS Command Line Interface (AWS CLI) to manage the processing of snapshots of an instance.


### Connecting to the Command Host EC2 instance

I use EC2 Instance Connect to connect to the "Command Host" EC2 instance. 

* **Step 10** On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the **EC2 Management Console**.

* **Step 11** In the navigation pane, choose **Instances**.

* **Step 12** From the list of instances, choose **Command Host**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b76c04ac-7dbb-4663-895e-da8871bdaf20)

* **Step 13** Choose **Connect**.

On the EC2 Instance Connect tab, choose Connect.

This option opens a new browser tab with the EC2 Instance Connect terminal window.

Note: If you prefer to use an SSH client to connect to the EC2 instance, see the guidance provided in the additional references section.

You use this terminal window to complete the tasks throughout the lab. If the terminal becomes unresponsive, refresh the browser or use the steps in this task to connect again.


### Taking an initial snapshot

I identify the EBS volume that's attached to the "Processor" instance and take an initial snapshot. To do so, I run commands in the EC2 Instance Connect terminal window. You can copy the command output to a text editor for subsequent use.


To display the EBS volume-id, run the following command: 


       aws ec2 describe-instances --filter 'Name=tag:Name,Values=Processor' --query 
       'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.{VolumeId:VolumeId}'



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8124443e-cff1-4a0a-b80f-6e7baefe1298)


Note: The command returns a response similar to this: "VolumeId": "vol-1234abcd". 

		You use this value for VolumeId throughout the lab steps when prompted.



21.  Next, you take snapshot of this volume. Prior to this, you shut down the "Processor" instance, which requires its instance ID.  Run the following command to obtain the instance ID:

    aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' --query 
     'Reservations[0].Instances[0].InstanceId'


     The command returns a value for INSTANCE-ID similar to this: "i-0b06965263c7ac08f"



![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5810690c-cd3e-49ed-96b4-d190af9cc228)


22. To shut down the "Processor" instance, run the following command and replace "INSTANCE-ID" with the instance-id that you retrieved earlier:

    aws ec2 stop-instances --instance-ids INSTANCE-ID


    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/416e899e-7e5d-491e-9cad-0f2654e53821)



    To verify that the "Processor" instance stopped, run the following command and replace "INSTANCE-ID" with your instance id.


    aws ec2 wait instance-stopped --instance-id INSTANCE-ID


    aws ec2 wait instance-stopped --instance-id i-0179ed69870b42fc1

    When the instance stops, the command returns to a prompt.

24.    To create your first snapshot of the volume of your "Processor" instance, run the following command and replace "VOLUME-ID" with the VolumeId that you retrieved earlier:

       aws ec2 create-snapshot --volume-id VOLUME-ID

aws ec2 create-snapshot --volume-id vol-0772600274d66bec8

   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0619fc30-e6b8-40f1-ad8e-3a686ebbf510)



25.   To check the status of your snapshot, run the following command and replace "SNAPSHOT-ID" with the SnapshotId that you retrieved earlier:

    aws ec2 wait snapshot-completed --snapshot-id SNAPSHOT-ID


26. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5eec8002-5f39-42f8-b806-70ef3406eed2)



## Scheduling the creation of subsequent snapshots


Using the Linux scheduling system (cron), you can set up a recurring snapshot process so that new snapshots of your data are taken automatically.

For the purposes of this lab, you schedule a snapshot creation every minute so that you can verify the results of your work.

In this task, you create a cron job to manage the number of snapshots that are maintained for a volume.

Note: This section of the lab doesn't stop the instance in order to create a large number of snapshots for the next step.


27. To create and schedule a cron entry that runs a job every minute, run the following command and replace "VOLUME-ID" with the VolumeId that you retrieved earlier:



28.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/64491388-55e0-44bd-88e2-dbc1dc9ab46a)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/bdcb3d0c-32e8-4a71-bcec-995e7916cbaf)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d8118301-0255-4f72-be5a-ea5b96ab8c34)


### Retaining the last two snapshots


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c3803207-a185-4c54-bd39-38a1f95bf143)


aws ec2 describe-snapshots --filters "Name=volume-id, Values=VOLUME-ID" --query 'Snapshots[*].SnapshotId'



33. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8356dd57-badb-499b-a367-3e2b8c8f8071)

34. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/61ff7e08-fcd3-4d8c-9b76-2801810ffb00)


### Challenge: Synchronize files with Amazon S3


36.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0ea2e6fe-7db2-4f37-b9bc-03ed70eeec94)


37. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/434d3e9c-b130-4402-8122-5aec100dd603)


39. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3f19a64f-647a-4c47-afc9-de1bbce9bc67)

40. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c072fc84-5d63-4b56-8f2d-d4a50c59c7b1)

42. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/cc2d36fa-81a8-4aec-9e00-977cf73febb5)

44. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b98a8a7f-47dd-4b3e-b6b2-7a411b8f8360)



45.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0791cd62-e52b-4960-83f1-868197e433ce)

46.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/143365ca-e16f-4419-8175-d1d90dfa3668)


47.![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f2f0193f-f3b0-442f-b535-854aeb50eff4)


48. ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6d75bfa4-a321-4d5c-9c0e-aee889c4658a)








