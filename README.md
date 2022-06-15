# Hướng dẫn cài đặt cấu hình GLPI trên Alam Linux 8 / Rocky 8
## 1. Cài đặt MariaDB  
  - Cài đặt curl  
```sh
sudo dnf install curl -y
```
  - Thêm repo MariaDB 10.7  
```sh
curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash -s -- --mariadb-server-version=10.7 --skip-maxscale --skip-tools
```
  - Cài đặt MariaDB 10.7  
```sh
sudo dnf -y install MariaDB-server MariaDB-client
```
  - Khởi động MariaDB cùng hệ thống  
```sh
sudo systemctl enable --now mariadb
```
Khi MariaDB đã khởi động chạy lệnh ``mariadb-secure-installation`` để cấu hình MariaDB  
```sh
sudo mariadb-secure-installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: <ENTER NEW PASSWORD>
Re-enter new password: <CONFIRM PASSWORD>
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
  - Tạo database và user cho GLPI  
```sh
mysql -u root -p

CREATE USER 'glpi_user'@'%' IDENTIFIED BY 'password';
GRANT USAGE ON *.* TO 'glpi'@'%' IDENTIFIED BY 'password';
CREATE DATABASE IF NOT EXISTS `glpi_db` ;
GRANT ALL PRIVILEGES ON glpi_db.* TO 'glpi_user'@'%';
FLUSH PRIVILEGES;
exit
```
  - Cấu hình timezone cho GLPI
```sh
sduo mysql_tzinfo_to_sql /usr/share/zoneinfo | sudo mysql -p -u root mysql
```
```sh
GRANT SELECT ON mysql.time_zone_name TO 'glpi_user'@'%';
FLUSH PRIVILEGES;
exit;
```
  - Cài đặt repo epel và yum-utils
```sh
sudo dnf install -y epel-release yum-utils
```
  - Cài đặt repo REMI cho Alam Linux 8
```sh
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```
  - Kích hoạt repo php 8.1
```sh
sudo dnf update -y
sudo dnf module enable php:remi-8.1 -y
```
  - Cài đặt php và các module cần thiết cho GLPI
```sh
sudo dnf install php php-cli php-common php-curl php-{gd,intl,libxml,mysqli,zlib,exif,ldap,openssl,zip,bz2}
```
  - Mở port web
```sh
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```
  - Cài đặt GLPI
```sh
wget https://github.com/glpi-project/glpi/releases/download/10.0.1/glpi-10.0.1.tgz
tar vxf glpi-10.0.1.tgz
```
  - Di chuyển thư mục glpi sang thư mục /var/www/
```sh
move glpi /var/www/
```
  - Đặt quyền cho thư mục 
```sh
sudo chmod -R 755 /var/www/glpi
sudo chown -R apache:apache /var/www/glpi
```
  - Bật một số boolean SELinux
```sh
sudo setsebool -P httpd_can_network_connect on
sudo setsebool -P httpd_can_network_connect_db on
sudo setsebool -P httpd_can_sendmail on
sudo chcon -R -t httpd_sys_rw_content_t /var/www/glpi/
```
  - Tạo tệp cấu hình httpd Apache:
```sh
sudo vim /etc/httpd/conf.d/glpi.conf
```
Nội dung sau:  

```sh
<VirtualHost *:80>
   ServerName glpi.example.com
   DocumentRoot /var/www/glpi

   ErrorLog "/var/log/httpd/glpi_error.log"
   CustomLog "/var/log/httpd/glpi_access.log" combined

   <Directory> /var/www/glpi/config>
           AllowOverride None
           Require all denied
   </Directory>
   <Directory> /var/www/glpi/files>
           AllowOverride None
           Require all denied
   </Directory>
</VirtualHost>
```
  - Khởi động lại service httpd
```sh
sudo systemctl enable httpd && sudo systemctl restart httpd
```
  - Sau đó, truy cập bảng điều khiển web GLPI để hoàn tất cài đặt GLPI  
```sh
http://glpi.example.com
```
  - Làm theo các bước cài đặt tiếp theo để hoàn tất thiết lập.
