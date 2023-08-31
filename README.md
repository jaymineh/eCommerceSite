# Introduction

This is a sample e-commerce application built for KodeKloud. The original deployment was outdated so I reployed it on my end and updated the steps. This was achieved on a LAMP stack.

Here's how to deploy it on CentOS/RHEL systems:

## Deploy Pre-Requisites

1. Install FirewallD

```
sudo yum update -y
sudo yum install -y firewalld
sudo service firewalld start
sudo systemctl enable firewalld
```

## Deploy and Configure Database

1. Install MariaDB

```
sudo yum install -y mariadb-server
sudo service mariadb start
sudo systemctl enable mariadb
```

2. Configure firewall for Database

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload
```

3. Configure Database

```
$ mysql
MariaDB > CREATE DATABASE ecomdb;
MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
MariaDB > FLUSH PRIVILEGES;
```

> ON a multi-node setup remember to provide the IP address of the web server here: `'ecomuser'@'web-server-ip'`

4. Load Product Inventory Information to database

Create the db-load-script.sql

```
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF
```

Run sql script

```

mysql < db-load-script.sql
```

5. Update Database bind address

Open the SQL config file found in `/etc/my.cnf` and insert the below line of code:

```
[mysqlnd]
bind-address = 0.0.0.0
```

*The above bind address allows for calls to the database from anywhere. Not recommended in PROD as this should be the private IP or subnet of the webserver.*

After updating the config file, run `systemctl restart mariadb`.

## Deploy and Configure Web

1. Install required packages

```
sudo yum install -y httpd php php-mysqlnd
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

2. Configure httpd

Change `DirectoryIndex index.html` to `DirectoryIndex index.php` to make the php page the default page

```
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

3. Start httpd

```
sudo service httpd start
sudo systemctl enable httpd
```

4. Test HTTPD

```
curl http://localhost
```

5. Download code

```
sudo yum install -y git
git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
```

6. Update index.php

Update [index.php](https://github.com/kodekloudhub/learning-app-ecommerce/blob/13b6e9ddc867eff30368c7e4f013164a85e2dccb/index.php#L107) file to connect to the right database server. In this case `localhost` since the database is on the same server.

Open the `index.php` file in your preferred editor and update this part of the code as seen below:

```

              <?php
                        $link = mysqli_connect('localhost', 'ecomuser', 'ecompassword', 'ecomdb');
                        if ($link) {
                        $res = mysqli_query($link, "select * from products;");
                        while ($row = mysqli_fetch_assoc($res)) { ?>
```

After modifying the `index.php` file, restart the database service so the new changes are taken into effect.

> ON a multi-node setup remember to provide the IP address of the database server instead of `localhost`.

7. Test

```
curl http://localhost
```

```
curl http://<PublicIP>:80
```

8. Visit Webpage

After checking on the terminal to see if the website is reachable, try to reach it via it's URL, which in this case would be the IP address of the server.

![Webpage](shopping.png)