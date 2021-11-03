# Project 1: Dựng Web Server chạy WordPress trên server linux (tự chọn Ubuntu hoặc CentOS)
Yêu cầu:
* Web chạy bằng nginx hoặc Apache
* Cấu hình ít nhất 2 virtual site
* Set Basic-Authen để 2 user vào được site
* Quản lý database bằng phpmyadmin và mysql CLI
* Cấu hình dùng nginx làm proxy của Apache (chạy chung trên 1 server)
* Config SSL HTTPS

Project này sử dụng CentOS 7

### Yêu cầu 1 + 2 : Web chạy bằng apache, cấu hình 2 virtual site
#### Bước 1: Cài đặt LAMP
- **Cài đặt Apache Web Server**

 Cài đặt Apache 
 
`# yum -y install httpd`

 Khởi động Apache
```
# systemctl enable httpd
# systemctl start httpd
```
Cấu hình firewall mở port cho dịch vụ http/https

```
# firewall-cmd --permanent --zone=public --add-service=http
# firewall-cmd --permanent --zone=public --add-service=https
# firewall-cmd --reload
```   
Truy cập địa chỉ ip của server (http://ip-server) để kiểm tra

- **Cài đặt MariaDB Database**

Cài đặt MariaDB

`# yum -y install mariadb-server mariadb`

Khởi động MariaDB
```
# systemctl enable mariadb
# systemctl start mariadb
```
Set password cho user root để đăng nhập

`# mysql_secure_installation`

- **Cài đặt PHP**

Cài đặt thêm 2 repo REMI và EPEL
```
# yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
# yum -y install epel-release yum-utils
```
Cài đặt PHP 7.3 và các gói bổ sung 
```
# yum-config-manager --enable remi-php73
# yum -y install php php-cli php-fpm php-mysqlnd php-zip php-devel php-gd php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json`
```
Kiểm tra phiên bản PHP vừa cài đặt

`# php -v`

#### Bước 2: Cấu hình 2 Virtual Host 

Trên server sẽ cấu hình chạy 2 website là web1.com và web2.com

Tạo Folder cho 2 website web1 và web2
```
# mkdir /var/www/web1
# mkdir /var/www/web2
```
Chỉnh sửa quyền truy cập với các file và thư mục bên trong /var/www

`# chmod -R 755 /var/www`

Tạo ra file index.html đơn giản cho 2 website

```
# touch /var/www/web1/index.html
# touch /var/www/web2/index.html
# echo "<center><h1>This is website web1.com</h1></center>" > /var/www/web1/index.html
# echo "<center><h1>This is website web2.com</h1></center>" > /var/www/web2/index.html
```
Tạo 2 thư mục lưu trữ File cấu hình Virtual host cho Apache:
```
# mkdir /etc/httpd/sites-available 
# mkdir /etc/httpd/sites-enabled 
```
Trong đó:
sites-available chứa các cấu hình Virtual host có trên hệ thống

sites-enabled chứa các cấu hình Virtual host được kích hoạt để chạy

Cấu hình để Apache nhận cấu hình những virtual host trong sites-enabled

` # vi /etc/httpd/conf/httpd.conf `

Thêm dòng sau vào cuối file, sau đó lưu lại và thoát

` IncludeOptional sites-enabled/*.conf `


Tạo File Virtual host cho web1.com

` # vi /etc/httpd/sites-available/web1.conf `

Thêm nội dung sau vào file, sau đó lưu lại và thoát
```
<VirtualHost *:80>
       ServerAdmin admin@web1.com
       ServerName web1.com
       ServerAlias www.web1.com
       DocumentRoot /var/www/web1
       DirectoryIndex index.php index.html
       ErrorLog /var/www/web1/error.log
       CustomLog /var/www/web1/requests.log combined
</VirtualHost>
```
Tương tự các bước như trên, ta tạo File Virtual host cho web2.com, thêm nội dung vào file, sau đó lưu lại và thoát

Kích hoạt Virtual host

Apache sẽ chỉ nhận những cấu hình Virtual host trong thư mục sites-enabled. vì vậy ta sẽ tạo một liên kết (symbolic link) vào thư mục sites-enabled 
```
# ln -s /etc/httpd/sites-available/web1.conf /etc/httpd/sites-enabled/web1.conf 
# ln -s /etc/httpd/sites-available/web2.conf /etc/httpd/sites-enabled/web2.conf 
```
Trỏ ip trong file hosts của windows để có thể phân dải 2 tên miền web1.com và web2.com về ip của Web server bằng cách vào Notepad với quyền admin, mở file host trong C:\Windows\System32\drivers\etc 

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/fih2550xmt_Screenshot%202021-10-05%20171917.png)

Disable SELINUX bằng cách đổi trạng thái SELINUX=disabled trong /etc/selinux/config

**Kiểm tra hoạt động của 2 virtual site bẳng cách truy cập web1.com và web2.com trên trình duyệt**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/gug54jsxqo_Screenshot%202021-10-05%20174906.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/a63gguvf2m_Screenshot%202021-10-05%20175012.png)


#### Bước 3: Cài đặt Wordpress

Ta sẽ cài đặt wordpress cho 2 trang web là web1.com và web2.com

- Đăng nhập vào mysql tạo database cho web1.com

` # mysql -u root -p `

```
MariaDB [(none)]> create database web1;
MariaDB [(none)]> create user userweb1@localhost identified by 'password';
MariaDB [(none)]> grant all privileges on web1.* to userweb1@locahost;
MariaDB [(none)]> flush privileges;
```
- Download wordpress và giải nén ra 
``` 
# yum -y install wget
# wget http://wordpress.org/latest.tar.gz 
# tar -xzvf latest.tar.gz 
```
- Cài đặt wordpress cho web1
```
# cp -r ~/wordpress/* /var/www/web1/ 
# mkdir /var/www/web1/wp-content/uploads 
# chown -R apache:apache /var/www/web1/* 
# cd /var/www/web1 
# cp wp-config-sample.php wp-config.php 
# vi wp-config.php 
```
Thay đổi trong file wp-config.php các thông tin về database như database name, database username, database Password. Sau đó lưu lại và thoát

```
/** The name of the database for WordPress */
define('DB_NAME', 'web1');
/** MySQL database username */
define('DB_USER', 'userweb1');
/** MySQL database password */
define('DB_PASSWORD', 'password');
```
> Thực hiện tương tự để cài đặt wordpress cho web2. Sau đó khởi động lại dịch vụ httpd bằng câu lệnh *# systemctl restart httpd*

**Truy cập web1.com và web2.com để xem kết quả cài đặt Wordpress**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/e88guhqb92_Screenshot%202021-10-05%20231907.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/wpxjy8v4ir_Screenshot%202021-10-05%20232308.png)

### Yêu cầu 3 : Set Basic-Authen để 2 user vào được site

#### Bước 1: Tạo user truy nhập httpd bằng lệnh htpasswd 

```
htpasswd -c /etc/httpd/conf/pwfile admin
New password :
Re-type new password :
```
#### Bước 2: Tạo file cấu hình auth_basic.conf
```
vi /etc/httpd/conf.d/auth_basic.conf
```
Thêm nội dung sau vào file

```
<Directory /var/www/web1/>
AuthType Basic
AuthName "Basic Authentication"
AuthUserFile /etc/httpd/conf/pwfile
Require valid-user
</Directory>

<Directory /var/www/web2/>
AuthType Basic
AuthName "Basic Authentication"
AuthUserFile /etc/httpd/conf/pwfile
Require valid-user
</Directory>
```
#### Bước 3:  Khởi động lại dịch vụ httpd 

` # systemctl restart httpd `

**Truy cập web1.com và web2.com để kiểm tra ta thấy Basic-Authen đã được set**

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/why9lzsh45_Screenshot%202021-10-05%20233837.png)

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/fyas1bcjep_Screenshot%202021-10-05%20234301.png)

### Yêu cầu 4: Quản lý database bằng phpmyadmin và mysql CLI

#### Quản lý bằng phpmyadmin

Cài đặt phpmyadmin 

` # yum -y install phpMyAdmin `

Chỉnh sửa file phpMyAdmin.conf, sau đó lưu và thoát

` # vi /etc/httpd/conf.d/phpMyAdmin.conf `


```
<Directory /usr/share/phpMyAdmin/>
        Options none
        AllowOverride Limit
        Require all granted
</Directory>
```
Truy nhập phpmyadmin trên trình duyệt để kiểm tra kết quả

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/iuo0bbk4gh_Screenshot%202021-10-05%20235650.png)

Tại đây ta có thể thao tác với các database

#### Quản lý bằng mysql CLI

Đăng nhập vào mysql

` # mysql -u root -p `

Tại đây ta có thể thao tác với các database

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/11dzb0okiz_Screenshot%202021-10-06%20000007.png)

### Yêu cầu 5: Cấu hình dùng Nginx làm proxy của Apache (chạy chung trên 1 server)

Ta sẽ cài đặt Nginx Reverse proxy và Apache Web Server trên cùng một server CentOS 7 có IP 192.168.1.67, Apache listen ở port 8080, Nginx listen ở port 80 

Disable SELINUX bằng cách đổi trạng thái SELINUX=disabled trong /etc/selinux/config

**Cài đặt và khởi động Apache Web Server**
```
# yum -y install httpd 
# systemctl enable httpd 
# systemctl start httpd 
```
```
# firewall-cmd --permanent --zone=public --add-port=80/tcp
# firewall-cmd --permanent --zone=public --add-port=8080/tcp
# firewall-cmd --reload
```
**Cấu hình Apache listen ở port 8080 thay vì 80 như mặc định**

` # vi /etc/httpd/conf/httpd.conf `

Đổi *Listen 80* thành *Listen 8080* sau đó lưu lại và thoát

Truy cập địa chỉ ip của server (http://ip-server:8080) để kiểm tra hoạt động của Apache

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/y5xlw8jaca_Screenshot%202021-10-06%20120634.png)

**Cài đặt và khởi động Nginx**
```
# yum -y install epel-release 
# yum -y install nginx 
# systemctl enable nginx 
# systemctl start nginx 
```
**Cấu hình Nginx làm proxy của Apache**

` # vi etc/nginx/nginx.conf `

 Thêm vào file nội dung sau, lưu lại và thoát 
 ```
location / {
    proxy_pass http://192.168.1.67:8080;
    proxy_set_header X-Forwarded-Proto  $scheme;
    proxy_set_header    Host            $host;
    proxy_set_header    X-Real-IP       $remote_addr;
    proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
```
Khởi động lại Nginx bằng lệnh ` # systemctl restart nginx `

**Kiểm tra kết quả**

Truy cập địa chỉ của Nginx trên trình duyệt ta sẽ được chuyển hướng tới website của Apache Web Server

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/rqz1gn4t66_Screenshot%202021-10-06%20122854.png)

### Yêu cầu 6: Config SSL HTTPS

Ở đây giả sử ta muốn cài đặt cho web1.com

Cài đặt Mod SSL

` # yum -y install mod_ssl `

Tạo thư mục để lưu private key và phân quyền
```
# mkdir /etc/ssl/private 

# chmod 700 /etc/ssl/private 
```
Tạo chứng chỉ SSL
```
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
Tạo virtual host tương tự như web1.com, tuy nhiên sử dụng port 443 (https)

` # vi /etc/httpd/sites-available/web1ssl.conf `

Thêm vào file nội dung sau, lưu lại và thoát
```
<VirtualHost *:443>
       SSLEngine on
       SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
       SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
       ServerAdmin admin@web1.com
       ServerName web1.com
       ServerAlias www.web1.com
       DocumentRoot /var/www/web1
       DirectoryIndex index.php index.html
       ErrorLog /var/www/web1/error.log
       CustomLog /var/www/web1/requests.log combined
</VirtualHost>
```

Khởi động lại dịch vụ httpd bằng lệnh ` # systemctl restart httpd ` 

Truy cập vào web1 với đường dẫn https://web1.com và kiểm tra kết quả

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/vsar3i7ws6_Screenshot%202021-10-06%20141752.png)

> Kết quả trên là do chứng chỉ SSL chúng ta tự tạo nên đó là chứng chỉ không hợp lệ 



