# icingaweb2
This is a step-by-step installation for Icinga Web 2 on CetOS 7


#Icinga Web 2 Version: 2.5.1
#PHP Version: 7.1
#MariaDB Connection Details:
#Database Name: icinga
#Database User: icinga
#Database Password: icinga
#Firewall and SELinux are disabled for this Installation
 


#Stop and Disable Firewalld and SELinux
 
systemctl stop firewalld && systemctl disable firewalld
nano /etc/selinux/config # change selinux to permissive
 
 
#Install EPEL Repo
 
yum install epel-release -y
 
 
#Install Apache and start/enable
 
yum install httpd -y
  
systemctl start httpd
systemctl enable httpd
 
 
#Install MariaDB, start & enable
 
yum install mariadb-server -y
  
systemctl start mariadb
systemctl enable mariadb
  
mysql_secure_installation
 
 
#Install the latest php packages for Icinga
 
yum install centos-release-scl -y
  
yum install rh-php71-php-mysqlnd rh-php71-php-cli rh-php71-php-common rh-php71-php-fpm rh-php71-php-pgsql rh-php71-php-ldap rh-php71-php-intl rh-php71-php-xml rh-php71-php-gd rh-php71-php-pdo rh-php71-php-mbstring -y
  
 
#set timezone to "Europe/Zurich"
nano /etc/opt/rh/rh-php71/php.ini
 
 
#start & enable php7
systemctl start rh-php71-php-fpm.service
systemctl enable rh-php71-php-fpm.service
 
 
#Install Icinga2 and icinga-ido-mysql, start/enable
 
rpm --import http://packages.icinga.org/icinga.key
rpm -i https://packages.icinga.org/epel/7/release/noarch/icinga-rpm-release-7-1.el7.centos.noarch.rpm
yum install icinga2 -y
yum install icinga2-ido-mysql -y
 
systemctl start icinga2.service
systemctl enable icinga2.service
 
 
#Create Icinga Database
 
mysql -u root -p
CREATE DATABASE icinga;
GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga.* TO 'icinga'@'localhost' IDENTIFIED BY 'icinga';
FLUSH PRIVILEGES;
EXIT;
 
 
#Import IDO Icinga schema & enable it
 
mysql -u root -p icinga < /usr/share/icinga2-ido-mysql/schema/mysql.sql
 
nano /etc/icinga2/features-available/ido-mysql.conf #Uncomment the "//" Lines
 
icinga2 feature enable ido-mysql
systemctl restart icinga2.service
 
icinga2 feature enable command
systemctl restart icinga2.service
 
 
#install icingaweb2 icingacli
yum install icingaweb2 icingacli -y
  
 
#Add 'apache' to icingacmd group
usermod -a -G icingacmd apache
  
 
#restart icinga2 service
systemctl restart icinga2.service
 
 
#create icinga token
icingacli setup token create
 
 
#Follow the instruction on the Site
#
#Authentication Type: Database
#Database Type: MySQL
#Port: 3306
#Database Name: icingaweb2
#Username: root
#Password: password
#
#Backend Name: icingaweb2
#
#Administration
#Username: icingaweb-admin
#Password: password
#
#Application Configuration
#Logging Type: File
#
#IDO Resource
#Port: 3306
#Database Name: icinga
#Username: icinga
#Password: icinga
#
#Command Transport
#Transport Type: Local Command File
