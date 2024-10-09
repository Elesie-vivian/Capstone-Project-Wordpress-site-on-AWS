# Wordpress Site on AWS

## OVERVIEW

This project implements a comprehensive WordPress solution for _digitalboost_, a small to medium-sized digital marketing agency using various AWS services.
We design a VPC, ensuring proper IP address range for the VPC, Public and Private subnets in two different Availability zones.
An Internet gateway is setup to allow communication between instances in the VPC  to connect to the internet.
We set up the web server and database servers in the private subnets to protect them.
Two route tables are set up; the public route table which we will associate with the public subnets and the private(main) route table which is associated with the private subnet.
We also configure Nat gateway to allow instances in the private subnets access the internet.
We will deploy a managed MySQL database using Amazon RDS for WordPress data storage.
An Elastic File System (EFS) will be used to store the WordPress files.
Lastly, we will configure an Application load balancer to distribute traffic among multiple instances as well as Auto-scaling groups to automatically scale instances based on the traffic load.
For all these we configure and implement appropriate security groups to ensure communication among these resources.
At the end of the project the final architecture will look like this.

![alt text](<Images/image 1.PNG>)