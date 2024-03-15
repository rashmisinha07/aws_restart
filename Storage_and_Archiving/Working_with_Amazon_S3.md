## Working with Amazon S3

### overview

I create and configure an Amazon Simple Storage Service (Amazon S3) bucket to share images with an external user at a media company (mediacouser) who has been hired to provide pictures of the products that the café sells. I also configure the S3 bucket to automatically send an email notification to the administrator when the bucket contents are modified.


The following diagram shows the component architecture of the Amazon S3 file-sharing solution and illustrates its usage flow.


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/dfa58e8f-688f-4d36-87a2-3752d1379726)

An AWS Identity and Access Management (IAM) user named mediacouser, which represents an external user at a media company, has been pre-created with the appropriate Amazon S3 permissions to allow the user to add, change, or delete images from the bucket. The necessary Amazon S3 permissions are reviewed for each user to make sure that access to the bucket is secure and appropriate for each role.  

The following steps describe the usage flow in the diagram:

1. When new product pictures are available or when existing pictures must be updated, a representative from the media company signs in to the AWS Management Console as **mediacouser** to upload, change, or delete the bucket contents.

2. As an alternative, **mediacouser** can use the AWS Command Line Interface (AWS CLI) to change the contents of the S3 bucket.

3. When Amazon S3 detects a change in the contents of the bucket, it publishes an email notification to the **s3NotificationTopic** Amazon Simple Notification Service (Amazon SNS) topic.

4. The administrator who is subscribed to the **s3NotificationTopic** SNS topic receives an email message that contains the details of the changes to the contents of the bucket. 

**Note:** In real-world implementations, external users might not receive direct access to CLI Host as depicted in the diagram.


## Connecting to the CLI Host EC2 instance and configuring the AWS CLI

I connect to the CLI Host EC2 instance by using EC2 Instance Connect and configure the AWS CLI so that I can run commands.

### Connecting to the CLI Host EC2 instance

I use EC2 Instance Connect to connect to the CLI Host EC2 instance. 

On the **AWS Management Console**, in the **Search** bar, enter and choose `EC2` to open the EC2 Management Console.

In the navigation pane, choose Instances.

From the list of instances, select the CLI Host instance.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c1ff3a33-6efe-4e8a-a0b8-60d759cdf6a6)

Choose Connect.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/138d6d6c-93f8-45d2-bbb8-7e1e627e895f)

On the EC2 Instance Connect tab, choose Connect.

This option opens a new browser tab with the EC2 Instance Connect terminal window.

You use this terminal window to complete the tasks throughout the lab. If the terminal becomes unresponsive, refresh the browser or use the steps in this task to connect again.


## Configuring the AWS CLI on the CLI Host instance

To set up the AWS CLI profile with credentials, run the following command in the EC2 Instance Connect terminal:

           aws configure
At the prompts, copy the following values that you pasted into your text editor, and paste them into the terminal window as directed.

AWS Access Key ID: Enter the value for AccessKey.

AWS Secret Access Key: Enter the value for SecretKey.

Default region name: Enter us-west-2.

Default output format: Enter json.

SecretKey	BhC706HyDCKvY9GRyJsDuQZzhPdoVfUPorP0Q057
LabRegion	us-west-2
AccessKey	AKIA6ODUZIVX3TKJLDWZ

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/73dfe2c9-09c1-4a2f-a1a9-9fbc5dab8fa1)

You are ready to run AWS CLI commands to interact with AWS services.

## Creating and initializing the S3 share bucket

I use the AWS CLI to create the S3 share bucket and upload a few images.

To do so, you run the following commands in the EC2 Instance Connect terminal window.

To create an S3 bucket, run the following command. In the command, replace <cafe-xxxnnn> with your bucket name. Your bucket name must begin with cafe- and should include a combination of letters and numbers to make your bucket name unique:

          aws s3 mb s3://<cafe-xxxnnn> --region 'us-west-2'
 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d1a6ba5e-32da-40a0-aefc-e42b0efe4670)
         
You should receive a message similar to the following: make_bucket: cafe-xxxx9999999

Note: Bucket names cannot contain uppercase letters. If you receive an error when you try to create your S3 bucket, make sure your bucket name doesn't include uppercase letters.

Next, you load some images into the S3 bucket under the /images prefix. Sample image files are provided in the initial-images folder on the CLI Host. 
15. To load images into the bucket, run the following command. In the command, replace <cafe-xxxnnn> with your bucket name:
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b9ec681a-f1ca-423c-af84-188f8baef0e7)


The command output lists the image files that are being uploaded.

To verify that the files were synced to the S3 bucket, run the following command. In the command, replace <cafe-xxxnnn> with your bucket name:

aws s3 ls s3://<cafe-xxxnnn>/images/ --human-readable --summarize


 
16![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1719fb16-7dd3-4ede-8d1f-790292c40bc6)

You see the details of the image files that were uploaded, including the number of files uploaded and the total size of the files.



### Reviewing the IAM group and user permissions

you review the permissions assigned to the mediaco IAM user group. This group was created to provide a way for the users of the media company to use the AWS Management Console or the AWS CLI to upload and modify images in the S3 share bucket. Creating the group makes it convenient to manage individual user permissions. You also review the permissions inherited by the mediacouser user that is part of the group.	

## Reviewing the mediaco IAM group
In this section, you review the permissions assigned to the mediaco group.

On the AWS Management Console, in the Search bar, enter and choose IAM to open the IAM Management Console.

In the navigation pane on the left, choose User groups.

From the User groups list, select mediaco.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ef999b42-be69-476a-bbc2-e1dbcc58e7d7)

The Summary page for the mediaco group is displayed.

Choose the Permissions tab.

Next to IAMUserChangePassword, choose + to expand the policy.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/de263a93-a49a-41e7-a67a-9ec26fe76f08)

If needed, review the AWS managed policy that permits users to change their own password.

To collapse the policy, choose -.

Next to mediaCoPolicy, choose + to expand the policy.

Note: You might have to scroll down to see the policy. 

Notice the following statements in this policy:

The first statement, identified by the Sid key name AllowGroupToSeeBucketListInTheConsole, defines permissions that allow the user to use the Amazon S3 console to view the list of S3 buckets in the account.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c8464f39-5c76-41a9-b82c-16512a7527b8)

The second statement, identified by the Sid key name AllowRootLevelListingOfTheBucket, defines permissions that allow the user to use the Amazon S3 console to view the list of first-level objects in the cafe bucket and other objects in the bucket.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/137607db-be0b-4a0f-a9a9-cdd76196794c)

The third statement, identified by the Sid key name AllowUserSpecificActionsOnlyInTheSpecificPrefix, defines permissions that specify the actions that the user can perform on the objects in the cafe-*/images/* folder. The main operations are GetObject, PutObject, and DeleteObject, which correspond to the read, write, and delete permissions that you want to grant to the mediacouser user. Two additional operations are included for eventual version-related actions.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f821dbda-1d64-4865-b128-8dcaab8793f1)



To collapse the policy, choose -.

## Reviewing the mediacouser IAM user

In this section, you review the properties of the mediacouser user.

In the IAM console navigation pane, choose Users.

From the Users list, select mediacouser.

On the Permissions tab, you should see two policies: IAMUserChangePassword and mediaCoPolicy. These policies are assigned to the mediaco IAM group that you reviewed in the previous task.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b7449117-c301-402d-b093-f8e550fab062)

To verify that you see the mediaco IAM group, choose the Groups tab. 

The mediacouser user is a member of this group and therefore inherits the permissions assigned to the mediaco group.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ca53dfed-fa84-40a2-b757-2daaf0dffa68)

Choose the Security credentials tab.

In the Access keys section, choose Create access key, and choose the following options:

Choose Command Line Interface (CLI).

Select the check box for I understand the above recommendation and want to proceed to create an access key.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/535a0d9c-5558-47c2-b583-e66bd1b88a9e)
Choose Next.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/35098ed8-c52b-4525-904b-d25b198a5960)

Choose Create access key.

The following message displays: Access key created
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/52168752-dbac-4bb0-9ea4-90deadc523da)

Choose Download .csv file.

Choose Done.

On the mediacouser page, from the Security credentials tab, copy the Console sign-in link. 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/428c55ee-da1a-41d3-81d3-6e74563d5f05)


## Testing the mediacouser permissions

you test the permissions that you have reviewed by signing in to AWS Management Console as mediacouser and performing the view, upload, and delete operations on the contents of the images folder in the S3 share bucket. These actions are the use cases that the external media company user is expected to perform on the bucket. In addition, you test the unauthorized use case, where the external user attempts to change the bucket permissions.

To sign in to the AWS Management Console as the mediacouser user, use one of the following options:

Important: Do not sign out of the session where you are signed in as the voclabs/user. Instead, choose one of two options:

Option 1: Use a different browser.

Option 2: Use the same browser type, but open a new incognito or private browser session. 

For either option that you choose, enter the Console sign-in link that you copied from the previous step into your new browser tab. The AWS Management Console sign-in page opens and already has the Account ID populated.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/69fbcde3-3ac9-4c87-8e2a-5206a99ba38c)![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b325645b-dd38-4e66-adc0-6b08517bea63)

On the sign-in page, enter the following credentials: 

Enter the following credentials:

IAM user name: mediacouser.

Password: Training1!.

Choose Sign in.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6fdebf76-aaa5-46df-9f3f-d20f98fd4faa)
On the new AWS Management Console page, in the Search bar, enter and choose S3 to open the S3 Management Console. 

From the list of buckets, select the bucket that you created earlier.

To display the list of images that were uploaded earlier, select images/.

To test the view use case, select Donuts.jpg, and choose Open.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f2d03894-7fd1-4157-86ea-577ac1e58bd4)

A new browser tab should open that shows a picture of various donuts.

Tip: If a new browser tab does not open, a banner or icon at the top of your browser will indicate that your browser is preventing the site from opening pop-up windows. Choose the banner or icon, and choose Allow pop-ups.

Close the browser tab that shows the Donuts.jpg image.

In the Console tab, in the breadcrumb trail at the top, choose images/ to see the contents of the images folder again.

To test the upload use case, choose Upload. 

On the Upload page, choose Add files, and choose any image or picture from your local computer.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/58ab9097-2e3c-46f7-b594-75ea3952cb0e)

Choose Upload.

To close the Upload: status page, choose Close.

Select the file that you uploaded, and choose Open.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1fb2801c-671b-4c91-90a4-d073439a4f4c)

A new browser tab should open that shows the file that you uploaded.

Close the browser tab that shows the file that you uploaded.

To test the delete use case, in the Console tab, in the image list, select the check box for Cup-of-Hot-Chocolate.jpg.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1b319b38-b33a-41e3-be5b-f76bbe506e28)

Choose Delete.

On the Delete objects page, in the Delete objects? box, enter delete.

Choose Delete objects. 

The object is deleted and no longer appears in the image list.

To close the Delete objects: status page, choose Close.

Next, you test the unauthorized use case where mediacouser attempts to change the bucket's permissions. 

In the breadcrumb trail at the top, choose your bucket to return to the bucket content list.

Choose the Permissions tab. 

This is where you can change a bucket's permissions. 

Notice that for Permissions overview, the following error message is displayed: "Insufficient permissions." mediacouser is prevented from changing the bucket permissions. You could also try to upload a file directly to the root of the bucket. This action should also fail.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5d63f5fb-bc99-4e20-be58-2cd45ebd1ff7)

Sign out of the Amazon S3 console as mediacouser.

You have successfully created an Amazon S3 bucket, and you have confirmed that it is securely configured for file sharing with another user.


## Configuring event notifications on the S3 share bucket
In this task, you configure the S3 share bucket to generate an event notification to an SNS topic whenever the contents of the bucket change. The SNS topic then sends an email message to its subscribed users with the notification message. Specifically, you perform the following steps:

Create the s3NotificationTopic SNS topic.

Grant Amazon S3 permission to publish to the topic.

Subscribe to the topic.

Add an event notification configuration to the S3 bucket.

## Creating and configuring the s3NotificationTopic SNS topic

Return to the AWS Management Console window where you are signed in as voclabs/user.

On the AWS Management Console, in the Search bar, enter SNS and choose Simple Notification Service to open the Simple Notification Service console.

If necessary, to open the navigation pane, choose the menu icon () on the left.

In the navigation pane, choose Topics.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/77e144db-66e6-47a8-ba2d-452ac3811a92)
Choose Create topic.

Choose Standard.

For Name, enter s3NotificationTopic.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c6c08148-5be8-4944-962a-4748ed8521e3)

Choose Create topic.

A message is displayed indicating that the s3NotificationTopic SNS topic has been successfully created.

From the s3NotificationTopic page in the Details section, copy and paste the ARN value to a text editor. You need this value later in this lab.
arn: **arn:aws:sns:us-west-2:992382371183:s3NotificationTopic**
To configure the topic's access policy, choose Edit.

Expand the Access policy - optional section.

Replace the contents of the JSON editor with the following policy. In the JSON object, replace <ARN of s3NotificationTopic> with the ARM value that you copied earlier, and replace <cafe-xxxnnn> with your S3 bucket name. Remember to remove the enclosing angle brackets (< >). 

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/705e7425-0da4-4704-9320-4d8818c15397)
Take a moment to review the intent of this policy. It grants the cafe S3 share bucket permission to publish messages to the s3NotificationTopic SNS topic.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/6a3f35da-09b5-4edb-8c15-43fc046337fa)

Choose Save changes.

Next, you subscribe to the topic to receive the event notifications from the S3 share bucket. 

In the s3NotificationTopic pane, choose the Subscriptions tab.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/11b330fd-4d31-46c6-a20e-8f079d54b7bc)

Choose Create subscription.

Choose the Topic ARN box, and choose the s3NotificationTopic SNS topic that appears as an option.

From the Protocol dropdown list, choose Email.

In the Endpoint box, enter an email address that you can access.

Choose Create subscription. 

A message displays that confirms that the subscription was created successfully.

Check the inbox for the email address that you provided. You should see an email message with the subject AWS Notification - Subscription Confirmation.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/26e82c3c-5f12-4555-9af4-464b7e4d33cd)

Open the email message, and choose Confirm subscription. A new browser tab opens and displays a page with the message Subscription confirmed!

## Adding an event notification configuration to the S3 bucket

In this task, you create an event notification configuration file that identifies the events that Amazon S3 will publish and the topic destination where Amazon S3 will send the event notifications. You then use the s3api CLI commands to associate this configuration file with the S3 share bucket.


In the terminal window for the CLI Host instance, enter the following command to edit a new file named s3EventNotification.json:

vi s3EventNotification.json
In the editor, to change to insert mode, press i.

In the following JSON object, replace <ARN of s3NotificationTopic> with the ARN value that you recorded earlier. Remember to remove the enclosing angle brackets (< >). Copy and paste your customized JSON configuration into the editor window. 

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/dccb935f-6420-4de3-9d55-15b50ff3b2d8)

Take a moment to review the intent of this configuration. It requests that Amazon S3 publish an event notification to the s3NotificationTopic SNS topic whenever an ObjectCreated or ObjectRemoved event is performed on objects inside an Amazon S3 resource with a prefix of images/.

Press ESC to exit insert mode.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b45b549d-884c-49d3-bfed-ac775683b4e2)

To save the file and exit the editor, enter :wq and press Enter.

To associate the event configuration file with the S3 share bucket, run the following command. In the command, replace <cafe-xxxnnn> with your S3 bucket name:

aws s3api put-bucket-notification-configuration --bucket <cafe-xxxnnn> --notification-configuration file://s3EventNotification.json
Wait a few moments, and then check the inbox for the email address that you used to subscribe to the topic. You should see an email message with the subject Amazon S3 Notification.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e9aa74b0-9576-4464-adda-897c85893f8a)

Open the email message, and examine the notification message. It should be similar to the following:

{"Service":"Amazon S3","Event":"s3:TestEvent","Time":"2019-04-26T06:04:27.405Z","Bucket":"","RequestId":"7A87C25E0323B2F4","HostId":"fB3Z...SD////PWubF3E7RYtVupg="}

Notice that the value of the "Event" key is "s3:TestEvent". Amazon S3 sent this notification as a test of the event notifications configuration that you set up.


 
## Testing the S3 share bucket event notifications

In this task, you test the configuration of the S3 share bucket event notification by performing the use cases that mediacouser expects to perform on the bucket. These actions include putting objects into and deleting objects from the bucket, which send email notifications. You also test an unauthorized operation to verify that it is rejected. You use the AWS s3api CLI command to perform these operations on the S3 share bucket.


To configure the CLI Host's AWS CLI client software to use the mediacouser credentials, in the SSH window for the CLI Host instance, enter the following command:

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3642f8a7-1606-4803-b3f5-be83827dc4be)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1c81b844-b024-45a2-a863-01b7362f7de5)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a602712a-22f3-4c75-ac32-fda5201a134d)


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8f73a8b4-3073-4612-8a48-7474f4ba219b)

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e5a684f5-ebd5-4203-ad35-db476a2187fc)











You use this link in the next task.

 
