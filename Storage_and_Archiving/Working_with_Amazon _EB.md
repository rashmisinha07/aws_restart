## Working with Amazon EBS
### overview

Amazon Elastic Block Store (Amazon EBS) is a scalable, high-performance block-storage service that is designed for Amazon Elastic Compute Cloud (Amazon EC2). In this lab, I learn how to create an EBS volume and perform operations on it, such as attaching it to an instance, creating a file system, and taking a snapshot backup.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/375b048d-dab9-4c21-9cde-7c6c4359ab53)

## Creating a new EBS volume

I create and attach an EBS volume to a new EC2 instance.

* **Step 1:**  On the **AWS Management Console**, in the **Search** bar, enter and choose EC2 to open the **EC2 Management Console**.

 * **Step 2:** In the left navigation pane, choose **Instances**.

  An EC2 instance named **Lab** has already been launched for my lab.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ccb3e101-2fd2-41bb-93f5-955ba0c1428c)

* **Step 3:**  Note the **Availability Zone** for the **Lab** instance. It looks similar to the following: **us-west-2a**

**Tip:** I might have to scroll to the right to see the **Availability Zone** column.

*  **Step 4:** In the left navigation pane, for **Elastic Block Store**, choose **Volumes**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7312b9c2-80a8-4b30-b186-c4503ccfb29a)

I see an existing (8 GiB) volume that the EC2 instance is using.

 * **Step 5:** Choose **Create volume**, and configure the following options:
   
   *  **Volume type**: Choose **General Purpose SSD (gp2)**.

   * **Size (GiB)**: Enter 1. 
  **Note:** You might be restricted from creating large volumes.

   * **Availability Zone:** Choose the same Availability Zone as your EC2 instance (which is us-west-2a in this case).
  
 * **Step 6:** In the **Tags -optional** section, choose **Add tag**, and configure the following options:

    * **Key:** Enter `Name`.

    * **Value:** Enter `My Volume`.

* **Step 7:**  Choose **Create volume**. 

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/45bb9c5c-8a8d-49ca-9be2-a5936918f438)

  A new volume appears with the status of Creating in the **Volume state** column. This status soon changes to Available. You might need to choose **Refresh**  to see your new volume.

**Tip:** You might have to scroll to the right to see the **Volume state** column.

## Attaching the volume to an EC2 instance
I attach my new volume to an EC2 instance.

* **Step 8:** Select **My Volume**.

* **Step 9:** From the **Actions** menu, choose **Attach volume**.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/7576af3f-fc4b-4d6c-93eb-1f5573d2f73d)

* **Step 10:** From the Instance dropdown list, choose the **Lab** instance.

The **Device name** field is set to **/dev/sdf**. Commands that you run later in this lab include this device identifier. 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a626b420-46b0-4da7-9d66-c5354ba2c2f8)

* **Step 11:** Choose **Attach volume**.

The **Volume state** of your new volume is now In-use.



## Connecting to the Lab EC2 instance
I  use EC2 Instance Connect to connect to the Lab EC2 instance. 

 * **Step 12:** On the **AWS Management Console**, in the **Search** bar, enter and choose EC2 to open the **EC2 Management Console**.

* **Step 13:** In the navigation pane, choose **Instances**.

* **Step 14:** From the list of instances, select the **Lab** instance.
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d6f82bcb-b13a-482c-86c8-346c137b7bef)

* **Step 15:** Choose **Connect**.

* **Step 16:** On the **EC2 Instance Connect** tab, choose **Connect**.

This option opens a new browser tab with the **EC2 Instance Connect** terminal window.

**Note:** If you prefer to use an SSH client to connect to the EC2 instance, see the guidance to Connect to Your Linux Instance.

You use this terminal window to complete the tasks throughout the lab. If the terminal becomes unresponsive, refresh the browser or use the steps in this task to connect again.


# Creating and configuring the file system

I add the new volume to a Linux instance as an ext3 file system under the /mnt/data-store mount point.

* **Step 17:** To view the storage that is available on your instance, in the EC2 Instance Connect terminal, run the following command:

           df -h

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/ee67a86c-f730-4d79-aacd-6b1d1a073934)

  These results show the original 8 GB disk volume. Your new volume is not yet shown.

* **Step 18:** To create an ext3 file system on the new volume, run the following command:

         sudo mkfs -t ext3 /dev/sdf

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3c9f1a97-4987-4ea8-a136-91be2ef6bf5f)


 
* **Step 19:** To create a directory to mount the new storage volume, run the following command:

         sudo mkdir /mnt/data-store



* **Step 20:** To mount the new volume, run the following command:
  
      sudo mount /dev/sdf /mnt/data-store
       echo "/dev/sdf   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab


![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/53c9a0ed-c290-4f94-a654-0d0bc0ce949b)

 The last line in this command ensures that the volume is mounted even after the instance is restarted.

 * **Step 21:** To view the configuration file to see the setting on the last line, run the following command:


        cat /etc/fstab
   
   ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/1de977dd-d9e6-41e2-a2ea-0d1abaf062fe)

  * **Step 22:**  To view the available storage again, run the following command:

      df -h

    ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a518083f-4480-45cd-80ea-d1d73f441d4f)

 * **Step 23:** To create a file and add some text on the mounted volume, run the following command:

        sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"

  * **Step 24:**  To verify that the text has been written to your volume, run the following command:


            cat /mnt/data-store/file.txt

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/d599343e-cd61-4515-ae54-c9a910b5d782)

## Creating an Amazon EBS snapshot
I  create a snapshot of your EBS volume.

Amazon EBS snapshots are stored in Amazon Simple Storage Service (Amazon S3) for durability. New EBS volumes can be created out of snapshots for cloning or restoring backups. Amazon EBS snapshots can also be shared among Amazon Web Services (AWS) accounts or copied over AWS Regions.


 * **Step 25:** On the **EC2 Management Console**, choose **Volumes**, and select **My Volume**.

  * **Step 26:** From the **Actions** menu, choose **Create snapshot**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/a2665cd1-c827-49a1-afe5-314b2a27cf44)


 * **Step 27:** In the **Tags** section, choose **Add tag**, and then configure the following options:

Key: Enter `Name`.

Value: Enter `My Snapshot`.

* **Step 28:** Choose **Create snapshot**.
 
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/09277b41-d52c-45ab-8f73-bbbd65939d3c)


* **Step 29:** In the left navigation pane, choose **Snapshots**.
  
   The **Snapshot status** of my snapshot is Pending. After completion, the status changes to Completed. Only used storage blocks are copied to snapshots, so empty blocks do not use any snapshot storage space.
  
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/f708da56-2952-496b-90b2-f2f4554d1be9)

 * **Step 30:** In  EC2 Instance Connect terminal window, to delete the file that I created on my volume, run the following command:

    sudo rm /mnt/data-store/file.txt

 * **Step 31:** To verify that the file has been deleted, run the following command:


    ls /mnt/data-store/file.txt
    
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/72ac59e0-7681-43dd-8829-fdee53957c18)


## Restoring the Amazon EBS snapshot

I need to retrieve data stored in a snapshot, I can restore the snapshot to a new EBS volume.


### Creating a volume by using the snapshot

* **Step 32:** On the **EC2 Management Console**, select **My Snapshot**.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/96d7d3d3-7752-49d0-abb0-226e6d6a468c)


* **Step 33:** From the **Actions** menu, choose **Create volume from snapshot**.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/39a06520-7471-4de1-b7a6-29dbce2962ea)

* **Step 34:** For **Availability Zone**, choose the same Availability Zone that you used earlier.
  
 * **Step 35:** In the **Tags - optional** section, choose **Add tag**, and then configure the following options:

   * **Key**: Enter `Name`.

   * **Value**: Enter `Restored Volume`.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/c7b61d6a-92f2-4b71-b206-52027bb08782)

* **Step 36:** Choose **Create volume**.

 * **Step 37:** To see my new volume, in the left navigation, choose **Volumes**.

The **Volume status** of my new volume is Available.

When restoring a snapshot to a new volume, I can also modify the configuration, such as changing the volume type, size, or Availability Zone.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/3d3f49c0-39dd-4bef-86fa-de840ae442fd)

## Attaching the restored volume to the EC2 instance

* **Step 38:** Select **Restored Volume**.
  
 * **Step 39:** From the **Actions** menu, choose **Attach volume**.
   
![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/85eba17a-0fc8-471f-a87f-8aa3e89cf4d9)

* **Step 40:**  From the **Instance** dropdown list, choose the **Lab** instance.

   The **Device name** field is set to **/dev/sdg**. You use this device identifier in a later task.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/e2587c38-5e20-405f-85f0-86ff83e7a2b1)

* **Step 41:** Choose **Attach volume**.
  
The **Volume status** of your volume is now In-use.

![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fd404f5b-6ac2-4014-b6aa-1bed978eb8a3)



## Mounting the restored volume

* **Step 42:** To create a directory for mounting the new storage volume, in the EC2 Instance Connect terminal, run the following command:

       sudo mkdir /mnt/data-store2

  To mount the new volume, run the following command:

      sudo mount /dev/sdg /mnt/data-store2


  To verify that the volume that you mounted has the file that you created earlier, run the following command:

         ls /mnt/data-store2/file.txt

  I see the file.txt file.

  ![image](https://github.com/rashmisinha07/aws_restart/assets/62481476/fce23a66-a5d4-4ffb-b006-d13c14f4853e)


  ## Conclusion


 I have successfully done the following:

* Created an EBS volume

* Attached and mounted an EBS volume to an EC2 instance

* Created a snapshot of an EBS volume

* Created an EBS volume from a snapshot



