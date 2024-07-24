---

# WordPress Website Deployment on AWS

This repository contains scripts and configuration files used to deploy a WordPress website on AWS infrastructure. Below is an overview of the setup and deployment process.

## Infrastructure Overview

The infrastructure setup on AWS includes the following components:

1. **Virtual Private Cloud (VPC)**:
   - Configured with public and private subnets across two availability zones for high availability.

2. **Internet Gateway**:
   - Facilitates connectivity between VPC instances and the wider Internet.

3. **Security Groups**:
   - Acts as a network firewall to control traffic to EC2 instances.

4. **Availability Zones**:
   - Utilized for enhanced system reliability and fault tolerance.

5. **EC2 Instances**:
   - Web servers (WordPress application servers) deployed in private subnets for enhanced security.

6. **NAT Gateway**:
   - Allows instances in private subnets to access the Internet.

7. **Application Load Balancer (ALB)**:
   - Distributes web traffic evenly across Auto Scaling Group instances in multiple Availability Zones.

8. **Auto Scaling Group**:
   - Automatically manages EC2 instances to ensure availability, scalability, fault tolerance, and elasticity.

9. **Amazon RDS (Relational Database Service)**:
   - Used for hosting the WordPress database for data storage and retrieval.

10. **Amazon EFS (Elastic File System)**:
    - Provides a shared file system for storing WordPress files and ensuring consistency across instances.

11. **Certificate Manager (ACM)**:
    - Secures application communications with SSL/TLS certificates.

12. **Route 53**:
    - Manages domain registration and DNS routing for the WordPress website.

13. **Simple Notification Service (SNS)**:
    - Sends notifications about Auto Scaling Group activities.

## Deployment Process

### Prerequisites

Before deployment, ensure you have the following:

- AWS CLI configured with necessary permissions.
- GitHub repository containing WordPress files for deployment.

### Deployment Steps

1. **Set Up Infrastructure**:
   - Use CloudFormation or AWS Management Console to create VPC, subnets, Internet Gateway, NAT Gateway, Security Groups, ALB, Auto Scaling Group, RDS, EFS, ACM certificates, Route 53 hosted zone, and SNS topics.

2. **Deploy EC2 Instances**:
   - Launch EC2 instances with Amazon Linux 2 AMI.
   - Use the provided script to install Apache HTTP Server and deploy WordPress files from GitHub repository.

   ```bash
   #!/bin/bash
   sudo su
   yum update -y
   yum install -y httpd
   cd /var/www/html
   wget https://github.com/Bertechaws/Host-HTML-site-via-aws/main.zip
   unzip main.zip
   unzip HTML-website-main/mole.zip
   cp -r mole-main/* /var/www/html
   rm -rf *.zip *.md HTML-website-main mole-main
   systemctl enable httpd
   systemctl start httpd
   ```

3. **Configure Auto Scaling**:
   - Set up Auto Scaling policies and CloudWatch alarms for scaling EC2 instances based on traffic.

4. **Database Configuration**:
   - Use RDS to create a MySQL database instance for WordPress.

5. **WordPress Setup**:
   - Complete WordPress installation and configuration using the AWS-hosted database.

6. **DNS Configuration**:
   - Create a Route 53 DNS record for the domain name pointing to the ALB.

7. **Testing and Validation**:
   - Test website functionality and ensure proper traffic distribution across instances.

### Accessing the Application

- Access the WordPress website via the domain name registered in Route 53.

### Security Considerations

- Ensure Security Groups and NACLs are properly configured to restrict access.
- Regularly update and patch EC2 instances and other AWS resources.






### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

```bash
# create to root user
sudo su

# update the software packages on the ec2 instance 
sudo yum update -y

# create an html directory 
sudo mkdir -p /var/www/html

# environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# mount the efs to the html directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 

# download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# restart the webserver
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

This script is included in the launch template for the Auto Scaling Group, ensuring that new instances are configured correctly with the necessary software and settings.

```bash
#!/bin/bash
# update the software packages on the ec2 instance 
sudo yum update -y

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y http

d
sudo systemctl enable httpd 
sudo systemctl start httpd

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
#
# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
#
# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# environment variable
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com

# mount the efs to the html directory 
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# set permissions
chown apache:apache -R /var/www/html

# restart the webserver
sudo service httpd restart
```

## How to Use

1. Clone this repository to your local machine.
2. Follow the AWS documentation to create the required resources (VPC, subnets, Internet Gateway, etc.) as outlined in the architecture overview.
3. Use the provided scripts to set up the WordPress application on EC2 instances within the VPC.
4. Configure the Auto Scaling Group, Load Balancer, and other services as per the architecture.
5. Access the WordPress website through the Load Balancer's DNS name.

## Contributing

Contributions to this project are welcome! Please fork the repository and submit a pull request with your enhancements.

## License

This project is licensed under the MIT License - see the LICENSE file for details.



## Support

For questions or assistance, please contact [Your Contact Information].

---

This README provides an overview of the deployment process and key components involved in hosting a WordPress website on AWS. Customize it further with specific details relevant to your setup and deployment preferences.
