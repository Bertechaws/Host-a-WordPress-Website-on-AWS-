![Alt text](2._Host_a_WordPress_Website_on_AWS.png)
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

## Support

For questions or assistance, please contact [Your Contact Information].

---

This README provides an overview of the deployment process and key components involved in hosting a WordPress website on AWS. Customize it further with specific details relevant to your setup and deployment preferences.
