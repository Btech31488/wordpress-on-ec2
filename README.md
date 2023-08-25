# WordPress Deployment on AWS EC2

A step-by-step guide to deploying WordPress on an AWS EC2 instance, covering EC2 setup, SSH access, and WordPress installation.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Create an EC2 Instance](#step-1-create-an-ec2-instance)
- [Step 2: SSH into Your EC2 Instance](#step-2-ssh-into-your-ec2-instance)
- [Step 3: Install WordPress and Its Dependencies](#step-3-install-wordpress-and-its-dependencies)
- [Step 4: Verify Installation](#step-4-verify-installation)
- [Conclusion](#conclusion)

## Prerequisites

- An AWS account.
- Basic knowledge of AWS EC2 and terminal commands.

## Step 1: Create an EC2 Instance

1. Log into your AWS account.
2. In the search bar, type "EC2" and click on the "EC2: link
3. Click on "Launch Instance" to create a new instance.
4. Select an Amazon Machine Image (AMI) - for this, you can choose the 'Amazon Linux 2023 AMI' which is free tier eligible
5. Choose an instance type. The "t2.micro" should suffice for basic purposes and it's free tier eligible.
6. Under "Key Pair (login)" Create a new Key pair, give it a name and download it. This will give you a '.pem' file which is used to SSH into the instance. Make sure it's stored securely.
7. Under "Network settings" click "Edit", keep the YPC, Subnet as the default. Make sure "Auto-assign public IP" is enabled. Create new a security group. Ensure you have rules that allow SSH (port 22) and HTTP (port 80). For the source, you can choose "Anywhere" but for the most secure method, only allow your IP address.
8. Under Configure, the default is 8 GiB, but you can adjust it if you like.
9. Click "Launch Instance"

## Step 2: SSH into Your EC2 Instance

1. Open your terminal (or powershell)
2. Navigate to the directory where you downloaded the ".pem" file
3. Run the following command to SSH into your instance:
   
```
ssh -i "your-key-name.pem" ec2-user@your-ec2-public-ip
```
## Step 3: Install WordPress and its dependencies

1. Install required packages

```
dnf install wget php-mysqlnd httpd php-fpm php-mysqli mariadb105-server php-json php php-devel -y
```

2. Download and Setup the latest WordPress package.

```
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
```
```
sudo cp -r wordpress/* /var/www/html/
```

3. Start Apache Server and Database Server:

```
sudo systemctl start mariadb httpd
```

4. Set up the database:
   - Secure the database
  
   ```
   sudo mysql_secure_installation
   ```
   - Type the current root password. By default, the root account does not have a password set. Press Enter.
      - Type Y to set a password, and type a secure password twice. For more information about creating a secure password, see https://identitysafe.norton.com/password-generator/. Make sure to store this password in a safe place.
      - Type Y to remove the anonymous user accounts.
      - Type Y to disable the remote root login.
      - Type Y to remove the test database.
      - Type Y to reload the privilege tables and save your changes.
   - Login to database server as the root user. Enter your database root password when prompted; this may be different than your root system password

   ```
   mysql -u root -p
   ```
   ```
   CREATE USER 'wordpress-user'@'localhost' IDENTIFIED BY 'your_strong_password';
   ```
   ```
   CREATE DATABASE `wordpress-db`;
   ```
   ```
   GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "wordpress-user"@"localhost";
   ```
   ```
   FLUSH PRIVILEGES;
   ```
   ```
   exit
   ```

5. Edit edit the wp-config.php file
   - Navigate to the WordPress directory
  
   ```
   cd /var/www/html/
   ```

   - Update the wp-config.php file:

   ```
   sudo vim wp-config.php
   ```

   - Find the line that defines DB_NAME and change database_name_here

   ```
   define('DB_NAME', 'wordpress-db');
   ```

   - Find the line that defines DB_USER and change username_here

   ```
   define('DB_USER', 'wordpress-user');
   ```

   - Find the line that defines DB_PASSWORD and change password_here

   ```
   define('DB_PASSWORD', 'your_strong_password');
   ```

   - Find the section called Authentication Unique Keys and Salts. These KEY and SALT values provide a layer of encryption to the browser cookies that WordPress users store on their local machines. Basically, adding long, random values here makes your site more secure. Visit https://api.wordpress.org/secret-key/1.1/salt/ to randomly generate a set of key values that you can copy and paste into your wp-config.php file.

5. Give the right permissions:

```
sudo chown -R apache /var/www
```
```
sudo chgrp -R apache /var/www
```
```
sudo chmod 2775 /var/www
find /var/www -type d -exec sudo chmod 2775 {} \;
```
```
find /var/www -type f -exec sudo chmod 0644 {} \;
```

6. Restart the Apache Web Server

```
sudo systemctl restart httpd
```
## Step 4: Verify Installation

1. In your browser, navigate to http://your-ec2-public-ip. This will take you to the WordPress setup page where you can complete the installation by choosing a username, password, and site name.

Remember: Always ensure your AWS services are secured appropriately. The steps above are for a basic deployment and do not include all best practices for a production environment. Consider using AWS RDS for database, AWS security groups, VPCs, etc., for better security and performance.

## Conclusion

You should now have a running WordPress instance on AWS EC2. Always ensure to follow best security practices when deploying applications in production environments.
