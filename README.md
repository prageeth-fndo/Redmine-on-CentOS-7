# Redmine-on-CentOS-7
Guide to install Redmine 5 on CentOS 7.

This guide will help you to properly install Redmine 5 on CentOS 7 distribution with MySQL 5.7, Ruby 3.0 and Apache 2.6. <br><br>
In this guideline, SOURCE INSTALLATION was done for the MySQL 5.7. <br>

### INITIAL SETUP
```
Yum update -y
```
### MYSQL INSTALLATION AND CONFIGURATION
```
yum groupinstall "Development Tools" -y
yum install cmake ncurses-devel -y
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.20.tar.gz
tar -xzf mysql-boost-5.7.20.tar.gz
cd mysql-5.7.20/
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr/local/mysql -DWITH_BOOST=/root/mysql-5.7.20/boost/
make
make install
cd /usr/local/mysql/
vim my.cnf
useradd mysql -s /sbin/nologin
chown -R mysql.mysql /usr/local/mysql/
bin/mysqld --initialize --user=mysql
bin/mysqld_safe --user=mysql & 
bin/mysql -u root -p -S /usr/local/mysql/mysql.sock
ALTER USER ' root' @' localhost'  IDENTIFIED BY 'root';
cp /root/mysql-5.7.20/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
ps aux | grep mysql
kill -9 PIDs
service mysqld start
service mysqld status
```
> change my.cnf path to /tmp/mysql.sock
```
chkconfig --level 345 mysqld on
export PATH=$PATH:/usr/local/mysql/bin
```
### DATABASE CREATION
```
mysql -u root -p
CREATE DATABASE redmine CHARACTER SET utf8mb4;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'redmine@123';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```
### RUBY INSTALLATION
```
yum install gcc-c++ patch readline readline-devel zlib zlib-devel libffi-devel  openssl-devel make bzip2 autoconf automake libtool bison sqlite-devel -y
curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
curl -L get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
rvm reload
rvm requirements run
rvm install 2.7
rvm list
ruby --version
```
### REDMINE INSTALLATION
```
wget https://www.redmine.org/releases/redmine-5.0.3.tar.gz
tar -xvf redmine-5.0.3.tar.gz
cd redmine-5.0.3
cp config/database.yml.example config/database.yml
nano config/database.yml
```
> change database.yml fie as follows
```
production:
adapter: mysql2
database: redmine
host: localhost
username: redmine
password: "redmine@123"
```
### DATABASE INTEGRATION
```
gem install bundler
bundle install --without development test
rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
RAILS_ENV=production REDMINE_LANG=en bundle exec rake redmine:load_default_data
```

### APACHE CONFIGURATION
```
yum install httpd
systemctl enable httpd
mv /root/redmine-5.0.3 /var/www/redmine
nano /etc/sysconfig/selinux 
set SELINUX=disabled
reboot
gem install passenger 
yum install curl-devel httpd-devel -y
passenger-install-apache2-module 
```
> copy \<IfModule> section
```
nano /etc/httpd/conf.d/passenger.conf 
```
> paste \<IfModule> section

> copy and paste following segement after \<IfModule> section
```
Listen 3000 
<VirtualHost *:3000> 
ServerName localhost 
DocumentRoot /var/www/redmine/public 
CustomLog "logs/redmine_access.log" combined 
ErrorLog "logs/redmine_error.log" 
<Directory /opt/redmine/public> 
Options -MultiViews 
AllowOverride all 
Require all granted 
</Directory> 
</VirtualHost>
```
### FIREWALL PERMISSION
```
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
sudo firewall-cmd --reload
```
> Or you can disable the firewall
```
systemctl disable firewalld
```
### OLD DB RESTORING
```
Drop the existing redmine database and create empty redmine database.
mysql -u root -p redmine < redmine-backup.sql
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
bundle exec rake db:migrate RAILS_ENV=production
bundle exec rake tmp:cache:clear
changing default password of admin
update users set hashed_password = '353e8061f2befecb6818ba0c034c632fb0bcae1b', salt ='' where login = 'admin';
hashed password is *password*
```
