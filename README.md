# Redmine-on-CentOS-7
Guide to install Redmine 5 on CentOS 7.

This guide will help you to properly install Redmine 5 on CentOS 7 distribution with MySQL 5.7, Ruby 3.0 and Apache 2.6. <br>
In this guideline, SOURCE INSTALLATION was done for the MySQL 5.7. <br>
You can always do YUM repository installation of the MySQL 5.7 with following commands. <br>
### Step 1 – Enable MySQL Repository 
```
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm 
```
### Step 2 – Installing MySQL 5.7 Server
```    
sudo yum install mysql-community-server 
```
### Step 3 – Initial Configuration
```
sudo systemctl start mysqld 
grep 'A temporary password' /var/log/mysqld.log |tail -1 
/usr/bin/mysql_secure_installation
```

> Complete the installation setup as follows

```
Securing the MySQL server deployment.

Enter password for user root: **********

The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration of the plugin.
Using existing password for root.

Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password: ******************

Re-enter new password: ******************

Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
```

### Step 4 – Login to MySQL
```
mysql -u root -p 
```
