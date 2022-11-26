# PROJECT 15 AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

## Created VPC
!["ACS-VPC"](images/ACS-vpc.png)

## Created Internet Gateway
!["ACS-InternetGW"](images/internet-gateway.png)

## Attached internet gateway to VPC
!["Attached GW"](images/attach%20vpc%20to%20igw.png)

## Created Subnet

- Private Subnet-1
!["private-subnet image"](images/access-points.pngprivate-subnet-1.png)

- Private Subnet-2
!["private-subnet2 image](images/private%20subnet2.png)

- Private Subnet-3
!["private-subnet3 image"](images/private-subnet3.png)

- Private Subnet-4
!["private-subnet4 image](images/private-subnet4.png)

- Public Subnet-1
!["public-subnet image"](images/public-subnet1.png)

- Public Subnet-2
!["public-subnet image2"](images/public-subnet2.png)

##  Creating Route Table
- Private Route table
!["private-route-table"](images/private-rtb.png)

- Public Route table
!["public-route-table"](images/public-rtb.png)

## Subnet Association with RTB
- Private Association
!["Private Association"](images/private%20association.png)

-Public Association
!["Public association"](images/)

## Creating NAT Gateway
- NAT Gateway
!["Nat-gateway"](images/NAT.png)

## Assocaiating NAT gateway to Private IP
- NAT gateway to Private IP
!["Private-IP Natgateway"](images/asscociating%20nat%20with%20private%20IP.png)


## Security Groups

- NGINX SERVER
!["Nginx Server Config"](images/nginx-reverse-proxy.png)
- BASTION SERVER
!["Bastion Server Config"](images/bastion.png)
- APPLICATION AND LOAD BALANCER
!["ALB Server Config"](images/int-Alb.png)

- WEB SERVER
!["Webserver Config"](images/webserver.png)
- DATA LAYER
!["DL Config"](images/datalayer.png)

## AWS certificate
![Aws-CERt](images/certificate.png)

## File system
!["AWS-filesytem"](images/filesystem.png)

## Created Access points
!["access-points"](images/access-points.png)

## ACS-RDS setup
- Acs rds
!["Acs rds"](images/ACS-rds.png)

-Acs-rds-subnet
!["ACS_rds-subnet"](images/ACS_rds-subnet.png)

-RDS database
![acs-rds-db](images/ACS-rds.png)

## Bastion ami installation
-------------------------------------
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch .rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum install wget vim python3 telnet htop git mysql net-tools chrony -y
systemctl start chronyd
systemctl enable chronyd

## Nginx ami installation 
-----------------------------------------
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

## webserver ami installation 
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

### setup for self-signed certificate for the nginx instance
```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
## setup of for self-signed certificate for the apache  webserver instance
```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt

vi /etc/httpd/conf.d/ssl.conf
```
## ACS AMI image
![Ami image](images/AMIs.png)

## ACS target groups
![target-groups](images/target-groups.png)

## ACS load balancer

![load-balancing](images/load-balancer.png)

## Launch template setup
### Bastion user-data
```
#!/bin/bash
yum install -y mysql
yum install -y git tmux
yum install -y ansible

```
### Nginx user-data
```
#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/Livingstone95/ACS-project-config.git
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config




```

### Wordpress user-data
```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0f9364679383ffbc0 fs-8b501d3f:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/ACSadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```
### Tooling user-data
```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-01c13a4019ca59dbe fs-8b501d3f:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/Livingstone95/tooling-1.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u ACSadmin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com', 'ACSadmin', 'admin1234', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

```