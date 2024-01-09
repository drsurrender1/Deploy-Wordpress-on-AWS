# Deploy-a-WordPress-Website-on-AWS
This is how I deploy my wordpress website on AWS based on tutorials provided by **Azeez Salu**.
### Lecture 1: Build a Three-Tier AWS Network VPC from Scratch
![flow chart1](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec1.png)
- Create VPC
- Edit DNS hostnames
- Attatch Internet Gateway to VPC
- Create 2 public subnets on 2 Availability Zones
- Enable auto-assign public IPv4 address for both subnets
- Create public route table
- Add routes to the public route table (simply connect the table to the Internet Gateway)
- Add subnet associations to the public route table. So it connects with the two public subnets.
- Create 4 private subnets
- Check main route table (default route table, created automatically). See if the four private subnets are associated with it.

### Lecture 2: Create Nat Gateways
![flow chart2](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec2.jpg)
- Create NAT Gateway in Public Subnet AZ1
- Create Private Route table AZ1
- Add routes to this route table, the target of this route table should be the NAT Gateway that we just created
- Add subnet associations to the route table we just created. So it connects with the two private subnets that reside in Availability Zone 1.
- Create another NAT gateway in Public Subnet AZ2
- Create Private Route Table AZ2
- Add routes to this route table, the target of this route table should be the **second** NAT Gateway that we just created
- Add subnet associations to the second route table we just created. So it connects with the two private subnets that reside in Availability Zone 2.

### Lecture 3: Create the Security Groups
- Create ALB Security Group for Application Load Balancer (Port = 80 and 443)
- Create SSH Security Group (Port = 22)
- Create Webserver Security Group (Port = 80 and 443, Source = ALB Security Group. Port = 22, Source = SSH Security Group)
- Create Database Security Group (Port = 3306, Source = Webserver Security Group)
- Create EFS Security Group for EFS File System

[Security group rules for different use cases](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-group-rules-reference.html)

### Lecture 4: Create the RDS Instance
![flow chart4](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec4.png)
- Before creating RDS database, first we need to create DB subnet group.
- Create database instance using RDS. Record your username and password for the database.

### Lecture 5: Create EFS
![flow chart5](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec5.png)
- Create an EFS file system
- During the creation step, remember to set Mount targets in two Private Data Subnets. So the webservers will use the mount
targets to  connect to the EFS

### Lecture 6: Install WordPress
![flow chart6](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec6.png)
- Launch EC2 instance on Public Subnet AZ1, call it Setup Server. (In the video, the author has created the key pair. If you do not have a key pari yet, you can create one in ppk format. **Don't create the key pair in pem format and converts it to ppk format, that would not work for some reason.**)
- Use PuTTY to connect to your instance using the key pair.
- Use Commands to install WordPress step by step
- Configure the wp-config.php file using your own database information
- Restart the webserver
- Enter the IPv4 address of EC2.You should see the WordDress form.
- Register and login account on WordPress

### Lecture 7: Create an Application Load Balancer
![flow chart7](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec7.png)
- Launch a EC2 instance on Private App Subnet AZ1. Enter [User Data](https://github.com/azeezsalu/wordpress-project-commands/blob/abaada4048b599a9b8d71f867f447a2304d9e385/8.%20Create%20an%20Application%20Load%20Balancer.txt). Make changes to the command about EFS. Call this instance Webserver AZ1.
- Launch a same EC2 instance on Private App Subnet AZ2. Call it Webserver AZ2.
- Create target group and name it Dev-TG. Register your target with Webserver AZ1 and Webserver AZ2.
- Create load balancer and name it Dev-ALB, mapping it to Public Subnet AZ1 and Public Subnet AZ2.
- Copy the DNS name of the Application Load Balancer. Login in the WordPress Dashboard, change the WordPress Address and Site Address.
- Terminate the setup server

### Lecture 8: Register a New Domain Name in Route 53
- Enter the Route 53 Dashboard, type the domain name and check its availability
- Purchase the domain name

### Lecture 9: Create a Record Set in Route 53
- Enter Route 53 Dashboard. Under the hosted zones, create the record
- Login in WordPress using the newly created record name, and change the URL

### Lecture 10: Register for an SSL Certificate in AWS Certificate Manager
- Create the public certificate.
- Create records in Route 53 for the certificate.

### Lecture 11: SSH into Instance in the Private Subnet
![flow chart11](https://github.com/Nina0917/Deploy-a-WordPress-Website-on-AWS/blob/main/mindmaps/lec11.png)
- Launch an EC2 instance in the public subnet AZ1, call it Bastion Host
- Download Pageant. Add your key pair to the pageant.
- Connect to the instance using Putty
- Access to other private subnet instance using ssh

### Lecture 12: Create an HTTPS Listener for the Application Load Balancer
- Find your current load balancer, and add listener
- Edit the listener of port 80. Redirect HTTP traffic to HTTPS.
- Connect to the Webserver AZ1 using SSH (what we learned in last lecture)。 Edit wp-config.php
- Check if the WordPress website is secure to connect or not
- Add wp-admin to the end of the domain name to enter the dashboard and change the URL

### Lecture 13: Create an Auto Scaling Group
- Terminate Webserver AZ1 and Webserver AZ2
- Create launch templates
- Create the auto scaling group
