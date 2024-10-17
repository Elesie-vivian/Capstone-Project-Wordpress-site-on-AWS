 # Capstone-Project-Wordpress-site-on-AWS
 ## Objective

  This project is a practical exploration of various AWS resources to design and implement a high performance WordPress solution which incorporates networking, compute, object solution and database in its design. 
  We will design this for _Digitalboost_, a small to medium-sized digital marketing agency to enhance its online presence.
  To achieve this, we will design and configure the following:

 * Virtual Private Cloud (VPC)
 * Public and Private subnets
 * AWS MySQL Relational Database (RDS) setup
 * EFS setup for WordPress files
 * Application Load Balancer
 * Auto scaling Group

  We will therefore split this project into six parts to extensively cover the  details of the setup.

 ### PART 1
 **VPC Setup**
  To setup VPC, first we will define the IP address range for the VPC.

  We determined a CIDR of 192.168.0.0/16 for our VPC.

  To determine the number of available IP address in the above CIDR block, we  use the following formular.
 
  2^(32 - CIDR notation) - 2
  =65,534.

  Therefore,65,534 possible IP addresses are available for the VPC.  
  We proceed to fill up necessary details to configure our VPC.

  We created and named a VPC: **WordPress vpc**

  ![alt text](<Images/Image 2.PNG>)

  ### PART 2
  **2.1, Public and Private Subnet Setup**

  We  will configure subnets within the VPC.

  We create a total of six(6) subnets, two(2) Public subnets in two different availability zones (us-east-1a and us-east-1b).
  The 2 Public subnets are setup for resources accessible from the internet.
  We then setup four(4) private subnets, two each within the above availability zones. 
  The private subnets are for resources with no direct internet access. Their CIDR blocks are also defined.

 ![alt text](<Images/Image 3.PNG>)


  **2.2, Creating Internet Gateway**

  Next, we create an Internet gatewy and attach it to the VPC. 
  The Internet Gateway successfully attached to the VPC allows resources in the VPC to connect to the internet.

 ![alt text](<Images/Image 4.PNG>)

 ![alt text](<Images/Image 5.PNG>)

  **2.3, Creating Route Tables**

  We will create two route tables;
  The Public route table and the Main route table.

 ![alt text](<Images/Image 6.PNG>)

  The Public route table is edited to route traffic to the internet gateway  thereby allowing connectivity to the internet.
 

  ![alt text](<Images/Image 7.PNG>)


  ![alt text](<Images/Image 8.PNG>)


  We associate the Public route table  with the two public subnets.


  ![alt text](<Images/Image 9.PNG>)


  **2.4, NAT Gateway Configuration**

  First, we create and associate an Elastic IP for NAT Gateway configuration, then create a NAT gateway for private subnet internet access.
  After creating the NAT Gateway, we update the route table of the private route table to attach the NAT Gateway to the private route table.
  This will allow instances in the private subnets to  access the internet.
  
 ![alt text](<Images/Image 10.PNG>)


  The Private (main) Route Table is then associated with the private subnets and routes traffic to the internet through the NAT Gateway.

  ![alt text](<Images/Image 11.PNG>)


  ![alt text](<Images/Image 12.PNG>)


  Next, we set up MySQL RDS to manage _DigitalBoost_ wordpress data in the cloud.


  ### PART 3
  **AWS MySQL RDS Setup**

  **3.1, RDS Instance Creation** 

  On the AWS console we search and locate RDS, then proceed to fill the relevant fields to setup RDS with MySQL engine for the wordpress site. 

  ![alt text](<Images/Image 13.PNG>)


  **3.2, Security Group Configuration**

  We will configure security group for the RDS instance and add an inbound rule for MySQL/Aurora on port 3306, allowing traffic from the 0.0.0.0/0 CIDR range. 
  This ensures that the security group attached to the database permits inbound traffic on port 3306.


  ![alt text](<Images/Image 14.PNG>)


  **3.3, WordPress-RDS Connection**
  
  Still on the AWS console, we locate EC2 and configure an Amazon Linux 2 Instance named Wordpress-App, to connect to our database.


  ![alt text](<Images/Image 15.PNG>)

  
  Now, we go further to configure our Amazon RDS database to connect to the EC2, by modifying our RDS database security group to allow network access to _WordPress-App_ EC2.
  Here for source we enter wordpress, we choose the wordpress security group that we used for our EC2 instance.
  
  ![alt text](<Images/Image 16.PNG>)

  Next, we SSH into our EC2 on the terminal and connect to the RDS database to create a database user for the WordPress application.
  
  **3.4, Creating a Database User**
 
  First, we run the following command to install MySQL client to interact with the database.

  sudo yum install -y mysql

  ![alt text](<Images/Image 17.PNG>)


  ![alt text](<Images/Image 18.PNG>)


  We also set the environment variable for MySQL host.

  export MYSQL_HOST=<your-endpoint>

  Note: We replace endpoint with the hostname of our RDS instance.


  ![alt text](<Images/Image 19.PNG>)


  Next, we run the following command to connect MySQL database. We replace 
  user and password with the master username and password we configured when creating the RDS database.

  mysql --user=<user> --password=<password> wordpress

  We then go ahead to create a database for our wordpress application and give the user permission to access the **wordpress**database.

  CREATE USER 'wordpress' IDENTIFIED BY 'wordpress-pass';
  GRANT ALL PRIVILEGES ON wordpress.* TO wordpress;
  FLUSH PRIVILEGES;
  Exit

  ![alt text](<Images/Image 20.PNG>)


  **3.5, Configuring WordPress on EC2**

  We will install the WordPress application and its dependencies on the EC2 instance so as to have a WordPress installation that is accessible on the web browser.

  * Installing the Apache web server.

    To run a Wordpress, we need to run a web server on our EC2 instance; we will use the open source Apache web server.
    We run the following command to install apache.

    sudo yum install -y httpd

    ![alt text](<Images/Image 21.PNG>)
    ![alt text](<Images/Image 22.PNG>)

  * To start the Apache web server: we run the following command

    sudo service httpd start

    ![alt text](<Images/Image 23.PNG>)

    We copy the Public IPV4 DNS of the EC2 instance and paste on the browser to verify that our Apache web server is working.

    ![alt text](<Images/Image 24.PNG>)

    

    * Download and configure WordPress

    We will now proceed to download and uncompress the software by running the following commands.

    wget https://wordpress.org/latest.tar.gz

    tar -xzf latest.tar.gz


    ![alt text](<Images/Image 25.PNG>)


    To view the contents of our directory called **wordpress** with the uncompressed contents, we run:

    $ ls

    ![alt text](<Images/Image 26.PNG>)

    We will change the directory to the wordpress directory and create a copy of the default config file using the following commands.

    cd wordpress
    cp wp-config-sample.php wp-config.php

    We will then open the wp-config.php file using the nano editor by running the following command

    nano wp-config.php

    ![alt text](<Images/Image 27.PNG>)

    Note: The Values imputed above are gotten from the earlier configurations made when we defined our user and password on the terminal.

    We save the above configuration and exit from nano.
    With the configuration updated we are on our way to deploy our website!

    **Deploying WordPress**

    In this step, we make our Apache web server handle requests for WordPress.
    We install the application dependencies needed for WordPress, we run the following command

    sudo amazon-linux-extras install -y mariadb10.5 php8.2

    ![alt text](<Images/Image 28.PNG>)

    Change to the proper directory with the command

    cd /home/ec2-user

    ![alt text](<Images/Image 29.PNG>)

    We then copy our WordPress application files into the /var/www/html directory used by Apache.

    sudo cp -r wordpress/* /var/www/html/

    Finally we restart the Apache web server to pick up the changes.

    sudo service httpd restart

 ![alt text](<Images/Image 30.PNG>)

 ![alt text](<Images/Image 31.PNG>)

 With this we have a publicly accessible WordPress installation using a fully managed MySQL database.

 ### PART 4

 **EFS Setup for WordPress Files**

 Here, we will utilize Amazon Elastic File System to store WordPress files for scalable and shared access.

 **4.1, Creating an EFS file system**
 On the AWS console we locate and click create file system.

 Our EFS is named **Wordpress-efs** 

 ![alt text](<Images/Image 32.PNG>)

 We will create two WordPress EC2 instances using Amazon Linux 2, with which to attach and mount our EFS.


![alt text](<Images/Image 33.PNG>)

 
 Next we'll create a security group to allow access from our EC2 instance to our EFS specifying the following:
 Type: NFS
 Port Range: 2049
 Source: 0.0.0.0/0


 ![alt text](<Images/Image 34.PNG>)


 **4.2, Mounting the EFS file system on WordPress instances**

 Back on the EFS page, we select the file system just created and click attach on the top right hand corner so as to mount our file system.

 Note: For regional EfS setup, EfS automatically created  a mount point for all availability zones.

 On our terminal ,SSH into our instance  and install the NFS client using 

  sudo yum -y install nfs -utils

  ![alt text](<Images/Image 35.PNG>)

  To start NFS service

  sudo service nfs start

  Then we check the status of the NFS service to ensure its running

  sudo service nfs status

  ![alt text](<Images/Image 36.PNG>)


  We switch to root user with

  sudo su

  then proceed to check the file system using command


  df -khP


  Create a folder (efs) on which to mount the EFS

  Using the NFS client, we copy the mount command to mount our files.


  ![alt text](<Images/Image 37.PNG>)


  Next, we check the file system again to confirm that the file is attached 
  
  df -khP

  ![alt text](<Images/Image 38.PNG>)


  and run the following command to enter into the folder created.


  cd /home/ec2-user efs2

  Next, run:

  pwd
  
  To check our current location.

  We will then develop contents in a file

  echo "hello world" > demo.txt and proceed to view the contents of the file with:


  cat demo.txt


 ![alt text](<Images/Image 39.PNG>)

  To mount the EFS in the second EC2 instance, we repeat the process above to check the file system and mount our files.


  ![alt text](<Images/Image 40.PNG>)



  ![alt text](<Images/Image 41.PNG>)

  We will add a new line in the file using vim text editor, this will also reflect in the first file mounted

  ![alt text](<Images/Image 42.PNG>)

  ![alt text](<Images/Image 43.PNG>)


  The above edit is also seen in the first file mounted earlier.

  ![alt text](<Images/Image 44.PNG>)

 * Configuring WordPress to use the shared file system

  To configure WordPress to use the shared file system the file permissions will be edited with 
  
  chmod 777 demo.txt

  ![alt text](<Images/Image 45.PNG>)


  We can see both instances using the same file system.


  ![alt text](<Images/Image 46.PNG>)


  ### PART 5
  **Application Load Balancer**

  In this section ,we will set up an Application Load Balancer to distribute incoming  traffic among multiple instances ensuring high availability and fault tolerance.

  On the AWS console we search and locate to create load balancer.
  We create an Application load Balancer called **wordpress-ALB**.

  Then, we create Target group to configure the load balancer.



  ![alt text](<Images/Image 47.PNG>)


  ![alt text](<Images/Image 48.PNG>)


  We will configure security group for our Load Balancer to allow HTTP traffic on Port 80.


  ![alt text](<Images/Image 49.PNG>)


  Note: we created two EC2 instances with basic content.



  ![alt text](<Images/Image 50.PNG>)



  ![alt text](<Images/Image 51.PNG>)


  **Configuring Listener Rules**

  We'll configure listener rules for routing traffic to instances.

  Note: The target group will be the two instances earlier created.



 ![alt text](<Images/Image 52.PNG>)

  After the Load Balancer is created we check the health of our instances.
  
 ![alt text](<Images/Image 53.PNG>)

 From the Load Balancer page, we now copy the DNS of the load balancer and paste on our web browser.


 ![alt text](<Images/Image 54.PNG>)



 Observe that the load balancer is evenly distributing traffic content across both instances created.


 ### PART 6

 **Creating Auto scaling Group**

 On the AWS console, we search and locate Auto scaling group and select "create auto scaling group".
 Next, click "create a launch template" and fill in the relevant fields to complete the creation


 We return to Auto scaling group page, choosing the launch template earlier created and specify the required instance attributes as 1 for minimum and 4 for maximum.

 We configure group size and scaling for launching instances for desired capacity, minimum desired capacity and maximum desired capacity.

 We then proceed to create the Auto scaling group.


 ![alt text](<Images/Image 55.PNG>)



 Next, we navigate to the instance management section to observe the behaviour of the Auto scaling group.

 To view this website, we copy the load balancer DNS and paste on the web.


  ![alt text](<Images/Image 56.PNG>)


  We observe the behaviour when one of the instances is deleted


  ![alt text](<Images/Image 57.PNG>)


  ![alt text](<Images/Image 58.PNG>)



Observe that the Auto Scaling Group will automatically launch new instances to maintain the desired state.



![alt text](<Images/Image 59.PNG>)



This therefore demonstrates the dynamic nature of ASG which adjusts the number of instances based on the configured desired settings.


**Notes on Troubleshhoot Issues:**

In the course of executing this project, some  troubleshoots needed to be resolved to successfully conclude the project.

* RDS Setup
 While creting a Database, encountered an error command

 _error establishing a database connection._

 to resolve this;

 I re-applied security group configuration by going back to the inbound rules section of my RDS security configuration, in the source section, ensuring its the wordpress security group, (which is the same as the EC2 sg) that is selected.

 ![alt text](<Images/Image 60.PNG>)

 Next, checked my connection to the RDS instance using the MySQL client.

 mysql -h [Endpoint] -P [Port] -u [username] -p[password]

 With the this done there was a successful connection to the RDS.


 * EFS setup

 While trying to mount my Efs using the NFS mount command, got an error msg

 Failed to resolve "fs-01f4a427341f6abf1.efs.us-east-1.amazonaws.com" name or server not known

 To resolve this, I checked all configurations and identified that the DNs hostname was not enabled.

 To resolve this :

 I went back to the AWS console, selecting the check box for the VPC, choosing **Actions** and then **Edit VPC**, and proceeded to select **Enable**.
With this the DNS hostname was enabled and the error resolved.


## Conclusion 

This capstone project has extensively demonstrated the WordPress architecture of _DigitalBoost_ marketing agency, detailing the network processes configured.

The various security measures implemented to ensure access are also explained in their various sections.

The WordPress site is shown after setup and we also showcased Auto Scaling Group by stimulating an unhealthy instance.

Some troubleshoot resolutions are detailed in the footnotes above which knowledge could be useful in subsequent projects.





 
 


  








  











    






 




