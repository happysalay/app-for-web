###########################################################################################
### mysql has already bean installed on CentOS, but mysql-devel is not
### install mysql-devel first, because php-fpm depends on it

yum install mysql-devel

########
### php7.0.12 install with fpm support
### part of next operation is refereced from http://php.net/manual/en/install.unix.nginx.php

##the install part
	cd php-7.0.12
  ./configure --help
## zlib is need(--with-zlib-dir), so download and install it first, http://www.zlib.net/
## mysql is already installed in /usr/bin, so the basedir is /usr, so ...
  ./configure --enable-fpm  --with-fpm-user=$USERNAME --with-fpm-group=$GROUPNAME --with-pdo-mysql=/usr --with-zlib-dir=/usr/local
  make 
  make install
##the configuration part, following installation
   cp ./sapi/fpm/php-fpm.conf /usr/local/etc/
   cp php.ini-development /usr/local/php/php.ini
   cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
##this next step is not used
   #cp sapi/fpm/php-fpm /usr/local/bin
##change include=NONE/etc/php-fpm.d/*.conf into  etc/php-fpm.d/*.conf in /usr/local/etc/php-fpm.conf
   sed -i'.bak' 's?NONE/etc/php-fpm.d?etc/php-fpm.d?' /usr/local/etc/php-fpm.conf
##copy config file
	 cd /usr/local/etc/php-fpm.d
	 cp www.conf.default www.conf
	 
##start php-fpm 
   php-fpm
   
   
   
##########
#### nginx
##########

##download pcre(version pcre-8.38.tar.gz, can't use latest)
##uncompress 
  gunzip -cd pcre-8.38.tar.gz |tar xvf -
##no need to comiple or install
 
##install nginx 1.18.0
  ./configure --with-pcre=../package/pcre-8.38
  make 
  make install
  
##config
  cd /usr/local/nginx/config
  vi nginx.conf
##uncomment the following lines, and replace /scripts with $document_root(don't know why yet)
  #location ~ \.php$ {
  #          root           html;
  #          fastcgi_pass   127.0.0.1:9000;
  #          fastcgi_index  index.php;
  #          #fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
  #          include        fastcgi_params;
  #      }
#like this
  location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
        
        
########
##mysql already installed, so basiclly I don't know how to install it
########
###/etc/my.cnf my look as follows:
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8


[mysqld]
collation-server=utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server=utf8

######
##create database using mysqladmin
######
##stop and start
 /etc/init.d/mysqld stop
 /etc/init.d/mysqld start
 
###create database
CREATE DATABASE `fanstecdb` CHARACTER SET utf8 COLLATE utf8_general_ci;
grant all privileges on fanstecdb.* to fanstec@'%' identified by 'fanstecdb';
