# Self-hosted WordPress on Nginx (Centos 7)

## Install needed packages and applications

Before you begin with this, there are a few steps that need to be completed first, You will need a CentOS 7 server installed and configured with a non-root user that has sudo privileges.

## Packages install
#### Nginx

sudo yum install epel-release
Step Two—Install Nginx

Now that the Nginx repository is installed on your server, install Nginx using the following yum command:

sudo yum install nginx
sudo systemctl start nginx
sudo systemctl enable nginx

If you are running a firewall, run the following commands to allow HTTP and HTTPS traffic:

sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload


Default Nginx-configuration at /etc/nginx/nginx.conf
Any additional server blocks, known as Virtual Hosts in Apache, can be added by creating new configuration files in /etc/nginx/conf.d. Files that end with .conf in that directory will be loaded when Nginx is started.

#### MySQL

Now, we will need to install MySQL and create a new DB for Wordpress.

sudo rpm -Uvh https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
sudo yum --enablerepo=mysql80-community install mysql-community-server
sudo service mysqld start
grep "A temporary password" /var/log/mysqld.log

Output is similar to: A temporary password is generated for root@localhost: hjkygMukj5+t783

sudo mysql_secure_installation
then enter the root password

Finish the installation then:

sudo service mysqld restart
mysql -u root -p

CREATE DATABASE wordpress;
CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
exit


#### PHP

sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum-config-manager --enable remi-php72
sudo yum install php72
sudo yum install php72-php-fpm php72-php-gd php72-php-json php72-php-mbstring php72-php-mysqlnd php72-php-xml php72-php-xmlrpc php72-php-opcache
php --version
sudo systemctl enable php72-php-fpm.service
sudo systemctl start php72-php-fpm.service
sudo systemctl status php72-php-fpm.service

#### WordPress
sudo yum install php-gd
sudo service httpd restart
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo rsync -avP ~/wordpress/ /var/www/html/ //or wherever you want, but make sure to remember this path to add it in the nginx file.

sudo chown -R nginx:nginx /var/www/html/* //make sure to edit the permissions to all files to nginx
sudo chmod 755 -R /var/www/html    //edit permissions to 755

Then copy the wp-config.php file and edit it to include your database

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');

Finish the installation by accessing the web interface on the server's ip/FQDN

## Configure Nginx with PHP

sudo vi /etc/opt/remi/php72/php-fpm.d/www.conf

Set user and group to nginx:
user = nginx
group = nginx

sudo systemctl restart php72-php-fpm.service

Update your nginx configuration file:
sudo vi /etc/nginx/conf.d/default.conf

sudo systemctl restart nginx





