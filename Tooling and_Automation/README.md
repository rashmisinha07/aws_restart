# Using AWS Systems Manager
AWS Systems Manager is a collection of capabilities that I can use to centralize operational data and automate tasks across your Amazon Web Services (AWS) resources. Systems Manager can configure and manage Amazon Elastic Compute Cloud (Amazon EC2) instances, on-premises servers, virtual machines, and other AWS resources at scale. 
# Generate inventory lists for managed instances
I can use Fleet Manager, a capability of Systems Manager, to collect operating system information, application information, and metadata from EC2 instances, on-premises servers, or virtual machines in a hybrid environment.I can also use Fleet Manager to query metadata to quickly understand which instances are running the software and configurations that your software policy requires and which instances I need update.

I used Fleet Manager to gather inventory from an EC2 instance.

* **Step-1** In the AWS Management Console, in the  search box, enter `Systems Manager` and press Enter. This option takes  to the Systems Manager console page.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/8b324433-f0fc-44ba-9bca-f8c0c19f6a9d)

* **Step-2** In the left navigation pane, for **Node Management**, choose **Fleet Manager**.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9eee8dff-9a76-489e-b780-e9d22638e1b7)

* **Step-3** Choose the Account management dropdown list, and choose Set up inventory.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/66832fb1-5cba-41f7-a9e2-5d7d04c5bf39)
* **Step-4** To create an association that collects information about software and settings for your managed instance, choose the following options:

   * In the **Provide inventory details** section, for **Name**, enter `Inventory-Association`
   * In the Targets section, choose the following options:
       * For Specify targets by, choose Manually selecting instances.
       * Select the row for Managed Instance.
Leave the other options as the default settings.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f19f9482-f6b6-4d6d-a6bb-c2da647f0771)

* **Step-5** Choose Setup Inventory
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/b8d25a70-827f-4e95-98fc-551a4d6f8f44)

* **Step-6** Choose the Node ID link, which directs to the Node overview.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/0f84347d-bc36-4f52-8521-ae49fbcf8bb5)

 * **Step-7** Choose the Inventory tab.

This tab lists all of the applications in the instance. Take a moment to review the installed applications and other options in the Inventory type dropdown list.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c0fa87d2-150f-4888-900a-8b6d83da9bf9)

I have successfully created a Systems Manager inventory association for your instance. Using Inventory, I can review and validate software configurations on your instances without needing to connect to each instance by using SSH.

# Install a Custom Application using Run Command

I install a custom web application (Widget Manufacturing Dashboard) by using Run Command, a capability of Systems Manager.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5f92963a-4d99-4611-b663-36fc621da0f7)

In the preceding diagram, Systems Manager installs an application on an EC2 instance within a virtual private cloud (VPC). It is installed by using Run Command. Run Command will run the "install script" and install the following: Apache web server, PHP, AWS SDK, and the web application. Once everything is installed, it also starts the web server.

* **Step-8** In the upper-left corner, expand the menu icon. For Node Management, choose Run Command.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/9eb5f9b6-5ace-41cf-8485-4c6a66bb47dd)

 
* **Step-9** Choose Run command
  A list appears of pre-configured documents for running common commands

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/08d5881c-9388-471d-8ecd-affb4175430d)

 * **Step-10** Choose the search icon  in the box, and a dropdown box appears. Choose the following options:
   
    * Owner
    * Owned by me
      
      A document appears.
      
      Note: Do not enter Owner or Owned by me. Entering this text does not return results.

 ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/681d1c9a-e7f6-40a0-905d-51cd196a94c0)

 * **Step-11** If the document is not already selected, select the button for the document.
 * **Step-12** The following information appears for this document:

    * Description Install Dashboard App
    * Document version: 1 (Default) 
   Leave the Document version option set to this default.
 * **Step-13** For Target selection, select Choose instances manually.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f6f0ad0b-d1f3-44b4-9944-7d8fbb8a56a8)

* **Step-14** In the Instances section, select Managed Instance.

The Managed Instance has the Systems Manager agent installed. The agent has registered the instance to the service, which allows it to be selected for Run Command.

 It is also possible to identify target instances by using tags. By using tags, I can run a single command on a whole fleet of matching instances.

 * **Step-14** In the Output options section, clear Enable an S3 bucket.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3ef96bfd-e7e6-45f4-8aef-50986e15e860)

  * **Step-15** Expand the AWS command line interface command section.

 This section displays the command line interface (CLI) command that initiates Run Command. I can copy this command and use it in the future within a script rather than having to use the AWS Management Console.

  * **Step-16** Choose Run

    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/4fe1b446-79cb-4564-a319-c3b3c06ba82f)

     A banner with the Command ID indicates that it was successfully sent on the Command ID page.

      * **Step-16**After 1–2 minutes, the Overall status should change to Success. If it doesn't, choose the  refresh icon to update the status.

      I now validate the custom application that was installed.


    * **Step-17** In the Vocareum console, choose the following options: 

  Choose the Details dropdown list at the top of these instructions.
   Choose Show
   Copy the ServerIP value. (This value is the public IP address.)

   * **Step-17** Open a new web browser tab, paste the IP address that you copied, and press Enter.

 The Widget Manufacturing Dashboard that you installed appears.

You have successfully used Run Command through Systems Manager to install a custom application onto your instance without needing to remotely access the instance by using SSH.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/217c716b-7cbe-4b21-a3da-8465b8f19d75)

# Use Parameter Store to manage application settings
Parameter Store, a capability of Systems Manager, provides secure, hierarchical storage for configuration data management and secrets management. You can store data such as passwords, database strings, and license codes as parameter values. You can store values as plain text or encrypted data. You can then reference values by using the unique name that you specified when you created the parameter.

I used Parameter Store to store a parameter that I used to activate a feature in an application.

* **Step-18** Keep the Widget Manufacturing Dashboard browser tab open, and return to the AWS Systems Manager tab.
* **Step-18** In the left navigation pane, for Application Management, choose Parameter Store.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/463868b9-775d-4849-b1e8-b3b09770834d)

 * **Step-18** Choose Create parameter
   * **Step-18**Choose the following options:

        * For Name:, enter /dashboard/show-beta-features
        * For Description: , enter Display beta features
        * For Tier:, leave the default option. 
        * For Type:, leave the default option. 
        * For Value:, enter True
   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/759d79be-3dc9-4fa2-82ae-76929533ccff)

  * **Step-18** Choose Create parameter

   A banner with the message "Create parameter request succeeded" appears at the top of the page. 

The parameter can be specified as a hierarchical path, such as /dashboard/<option>.

The application that is running on Amazon EC2 automatically checks for this parameter. If it finds this existing parameter, then additional features are displayed.



 * **Step-18** Return to the web browser tab that displays the application, and refresh the web page.
   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/55ae88ed-7058-418b-9d29-53869e40c8f4)

   Notice that three charts are displayed. The application is now checking Parameter Store to determine whether the additional chart (which is still in beta) should be displayed. It is common to configure applications to display "dark features" that are installed but not yet activated.

Optional: Delete the parameter, and then refresh the browser tab with the application. The third chart disappears again.

# Use Session Manager to access instances

With Session Manager, a capability of Systems Manager, you can manage your EC2 instances through an interactive one-step browser-based shell or through the AWS Command Line Interface (AWS CLI). Session Manager provides secure and auditable instance management without the need to open inbound ports, maintain bastion hosts, or manage SSH keys. You can also use Session Manager to help comply with corporate policies that require controlled access to instances, strict security practices, and fully auditable logs with instance access details while still providing end users with one-step cross-platform access to your EC2 instances.

When you use Session Manager with Microsoft Windows, Session Manager provides access to a PowerShell console on the instance.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d43fd3e3-7dcc-494c-855c-f936428e48b5)

In the preceding diagram, Systems Manager uses Session Manager to access the EC2 instance without having to connect to the instance by using SSH. Session Manager is one of the secure ways to access the instance.

I access the EC2 instance through Session Manager.

* **Step-18**In the left navigation pane, for Node Management, choose Session Manager.
* ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3b6735bf-3ba7-42b6-86b1-f483fca2222c)


* **Step-18**Choose Start session

* **Step-18**Select Managed Instance.
  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c04a36c8-f574-492e-b7cb-b69b827424ab)


* **Step-18**Choose Start session
A new session tab opens in your browser.

To activate the cursor, choose anywhere in the session window.

* **Step-18**Run the following command in the session window:

           ls /var/www/html

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ce724a6d-927b-44a9-a55f-f605512511da)

The output lists the application files that were installed on the instance.

* **Step-18**Run the following command in the session window:
  
                # Get region
                AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
                export AWS_DEFAULT_REGION=${AZ::-1}
                # List information about EC2 instances
                 aws ec2 describe-instances

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/5eac6d3a-f29b-47cc-8155-0afcd900e6bd)


The output lists the EC2 instance details for the Managed Instance in JSON format.

you can use Session Manager to log in to an instance without using SSH. You can also verify this capability by confirming that the SSH port is closed for the instance's security group.

You can restrict access to Session Manager through AWS Identity and Access Management (IAM) policies, and AWS CloudTrail logs Session Manager usage. These options provide better security and auditing than traditional SSH access.


   





   













