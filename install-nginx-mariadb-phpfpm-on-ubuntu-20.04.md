# How to Install Nginx, MariaDB, PHP-FPM on Ubuntu 20.04

This is a way to install and set up Nginx, MariaDB and PHP-FPM on Ubuntu 20.04.

> NOTE: This has been prepared for ease of use in mind, not security, mostly in development machine. Please do not use these instructions to setup on a public server environment. Use other proper manuals instead.

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
And if you can not login with error message like `Access denied for user 'root'@'localhost'` let's log in to mysql using sudo first

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
Now, you have access to account root.

## Installing PHP-FPM

Nginx uses PHP-FPM and for convenient, we use Ondrej Sury's PPA, so we can install multiple versions of PHP.

```
$ sudo apt install libapache2-mod-fcgid
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php && sudo apt update
$ sudo apt install -y php7.4 php7.4-fpm php7.4-curl php7.4-gd php7.4-json php7.4-mbstring php7.4-mysql php7.4-opcache php7.4-xml php7.4-xmlrpc php7.4-fileinfo php7.4-imagick php-pear
$ sudo service php7.4-fpm start
```
Check if PHP operated using Netstat:
```
$ netstat -pl | grep php
```
You're good to go if the result is similar to the text below:
```
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
unix  2      [ ACC ]     STREAM     LISTENING     30323    -                    /run/php/php7.4-fpm.sock
```

## Nginx and PHP-FPM Configuration
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
Save it and exit.

Test nginx config and make sure there's no error
```
$ sudo nginx -t
```
then restart the service
```
$ sudo systemctl reload nginx
```
## PHP-FPM Configuration

Go to /etc/php/7.4/fpm and edit php.ini file

Uncomment cgi.fix_pathinfo, change value to 0
```
cgi.fix_pathinfo=0
```
Save and exit.

Reload PHP-FPM service
```
$ sudo service php7.4-fpm restart
```
