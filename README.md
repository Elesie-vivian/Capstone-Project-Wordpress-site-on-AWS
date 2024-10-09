 # Capstone-Project-Wordpress-site-on-AWS
 ## Objective

 This project is a practical exploration of various AWS resources to design and implement a high performance WordPress solution which incorporates networking, compute, object solution and database in its design. We will design this for _Digitalboost_, a small to medium-sized digital marketing agency to enhance its online presence.
 To achieve this, we will design and configure the following:

 * Virtual Private Cloud (VPC)
 * Public and Private subnets
 * AWS MySQL Relational Database (RDS) setup
 * EFS setup for WordPress files
 * Application Load Balancer
 * Auto scaling Group

 We will therefore split this project into six parts to extensively cover the  details of the setup

 ### PART 1
 **VPC Setup**
 To setup VPC, first we will define the IP address range for the VPC

 We will use a CIDR of 192.168.0.0/16 for our VPC.
 To determine the number of available IP address in the above CIDR block using the following
 
 2^(32 - CIDR notation) - 2
 =65,534.
 Therefore,65,534 possible IP addresses are available for the VPC. We proceed to fill up necessary details to configure our VPC.
 We create and named a VPC: WordPress vpc

 ![alt text](<Images/Image 2.PNG>)

 ### PART 2
 **Public and Private Subnet Setup**

 We  will configure subnets within the VPC.
 We create a total of six(6) subnets, two(2) Public subnets in two different availability zones (us-east-1a and us-east-1b).The 2 Public subnets are setup for resources accessible from the internet.
 We then setup four(4) private subnets, two each within the above availability zones.The private subnets are for resourceswith no direct internet access. We also defined their CIDR blocks.

 ![alt text](<Images/Image 3.PNG>)


 **Creating Internet Gateway**

 Next we create an Internet gatewy and attach it to the VPC. The Internet Gateway successfully attached to the VPC allows resources in the VPC to connect to the internet.

 ![alt text](<Images/Image 4.PNG>)

 ![alt text](<Images/Image 5.PNG>)

 **Creating Route Tables**
  We create two route tables;
  The Public route table and the Main route table.

 ![alt text](<Images/Image 6.PNG>)

 We edit the Public route table to route traffic to the internet gateway  thereby allowing connectivity to the internet.
 

  ![alt text](<Images/Image 7.PNG>)

  ![alt text](<Images/Image 8.PNG>)

  We will associate the Public route table  with the two public subnets

  ![alt text](<Images/Image 9.PNG>)

  **NAT Gateway Configuration**

  First, we create and associate an Elastic IP for NAT Gateway configuration, then create a NAT gateway for private subnet internet access.
  After creating the NAT Gateway, we update the route table of the private route table to attach the NAT Gateway to the private route table.
  This will allow instances in the private subnets to  access the internet.
  
 ![alt text](<Images/Image 10.PNG>)


  The Private (main) Route Table is then associated with the private subnets and routes traffic to the internet through the NAT Gateway.

  ![alt text](<Images/Image 11.PNG>)

  ![alt text](<Images/Image 12.PNG>)

  Next we set up MySQL RDS to manage _DigitalBoost_ wordpress data in the cloud.

  ### PART 3
  **AWS MySQL RDS Setup**

  **3.1, RDS Instance Creation** 

  On the AWS console we search and locate RDS, then proceed to fill the relevant fields to setup RDS with MySQL engine for the wordpress site. 

  ![alt text](<Images/Image 13.PNG>)


  **3.2, Security Group Configuration**

  We will configure security group for the RDS instance and add an inbound rule for MySQL/Aurora on port 3306, allowing traffic fom the 0.0.0.0/0 CIDR range. 
  This ensures that the security group attached to the database permits inbound traffic on port 3306.

  ![alt text](<Images/Image 14.PNG>)

  **3.3, WordPress-RDS Connection**
  
  Still on the AWS console, we locate EC2 and configure an Amazon Linux 2 Instance named Wordpress-App, to connect to our database.

  ![alt text](<Images/Image 15.PNG>)
  
  Now we go further to configure our Amazon RDS database to connect to the EC2, by modifying our RDS database security group to allow network access to _WordPress-App_ EC2.
  Here for source we enter wordpress, we choose the wordpress security group that we used for our EC2 instance.
  
  ![alt text](<Images/Image 16.PNG>)

  Next, we SSH into our EC2 on the terminal and connect to the RDS database to create a database user for the WordPress application.
  
  **3.4, Creating a Database User**
 
  First we run the following command to install MySQL client to interact with the database.

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

  * Installing the Apache web server
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

    We will now proceed to download and uncompress the software by running the following comands.

    wget https://wordpress.org/latest.tar.gz
    tar -xzf latest.tar.gz

    ![alt text](<Images/Image 25.PNG>)

    To view the contents of our directory called **wordpress** with the uncompressed contents, we run:

    $ ls

    ![alt text](<Images/Image 26.PNG>)

    We will change the directory to the wordpress directory and create a copy of the default config file using the following commands

    cd wordpress
    cp wp-config-sample.php wp-config.php

    We will then open the wp-config.php file using the nano editor by running the following command

    nano wp-config.php

    ![alt text](<Images/Image 27.PNG>)

    Note: The Values imputed above are gotten from the earlier configurations made when we defined our user and password on the terminal.

    We save the above configuration and exit from nano.
    With the configuration updated we are on our way to deploy our website!

    **Deploying WordPress**






    






 




