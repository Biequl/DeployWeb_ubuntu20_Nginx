# How to Install Nginx, MariaDB, PHP-FPM on Ubuntu 20.04

```
$ sudo apt update
```
## Nginx
```
$ sudo apt install nginx -y

$ sudo systemctl start nginx

$ sudo systemctl enable nginx
```
## Firewall Setup
```
$ sudo ufw allow ssh
$ sudo ufw allow http
$ sudo ufw enable
```
## MariaDB
```
$ sudo apt install mariadb-server -y
$ sudo mysql_secure_installation
```
Follow the instruction to what you need. After it ends, try to login:
```
$ sudo mysql -u root -p
```
And if you can not login with error message `Access denied for user 'root'@'localhost'`

```
$ sudo mysql -u root -p
```
In order to create a password, first we have to hash the string first:
```
SELECT PASSWORD('verysecretchangeit');
```
For example we get this hashed string: `*54958E764CE10E50764C2EECBB71D01F08549980`

```
ALTER USER root@localhost IDENTIFIED BY PASSWORD '*54958E764CE10E50764C2EECBB71D01F08549980';
```
Restart MariaDB service
```
$ sudo service mariadb restart
```
## Installing PHP-FPM
```
$ sudo apt install libapache2-mod-fcgid
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php && sudo apt update

$ sudo apt install -y php7.4 php7.4-fpm php7.4-curl php7.4-gd php7.4-json php7.4-mbstring php7.4-mysql php7.4-opcache php7.4-xml php7.4-xmlrpc php7.4-fileinfo php7.4-imagick php-pear

$ sudo service php7.4-fpm start
```
Check if PHP operated:
```
$ netstat -pl | grep php
```
## Nginx and PHP-FPM configuration
```
$ vi /etc/nginx/nginx.conf
```
Uncomment the following lines.
```
keepalive_timeout 2;
server_tokens off;
```
Save the config file.
```
$ vi sites-available/default
```
Add or change this line:
```
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
}
```
Save and exit.

Test nginx config and make sure there's no error, then restart the service
```
$ sudo nginx -t
$ sudo systemctl reload nginx
```
## PHP-FPM Configuration

Go to /etc/php/7.3/fpm and edit php.ini file

Uncomment cgi.fix_pathinfo, change value to 0
```
cgi.fix_pathinfo=0
```
Save and exit.

Reload PHP-FPM service
```
$ sudo service php7.4-fpm restart
```
