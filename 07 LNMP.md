# 安装PHP+Nginx+MySQL

@author jiancaigege@163.com
@date 2016-6-5 14:28:02

[TOC]

---

本文以centos6为例。

## 安装PHP
### 下载

```
http://cn2.php.net/distributions/php-5.6.22.tar.bz2
http://cn2.php.net/distributions/php-7.0.7.tar.bz2
```

### 更新yum源
这里将Centos的yum源更换为国内的阿里云源。yum安装正常的可以跳过本步骤。

阿里云Linux安装镜像源地址：
http://mirrors.aliyun.com/

1、备份你的原镜像文件，以免出错后可以恢复：
``` shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

2、下载新的CentOS-Base.repo 到`/etc/yum.repos.d/`
``` shell
## CentOS 5
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo

## CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

## CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

3、生成缓存
``` shell
yum clean all
yum makecache
```

### 安装依赖
``` shell
yum install -y gcc gcc-c++ make cmake bison autoconf wget lrzsz
yum install -y libtool libtool-ltdl-devel 
yum install -y freetype-devel libjpeg.x86_64 libjpeg-devel libpng-devel gd-devel
yum install -y python-devel  patch  sudo 
yum install -y openssl* openssl openssl-devel ncurses-devel
yum install -y bzip* bzip2 unzip zlib-devel
yum install -y libevent*
yum install -y libxml* libxml2-devel
yum install -y libcurl* curl-devel 
yum install -y readline-devel
```

需要编译libmcrypt、mhash、mcrypt库（下载见文末附件）
``` shell
tar zxvf /libmcrypt-2.5.8.tar.gz \
&& cd /libmcrypt-2.5.8 && ./configure && make && make install && cd - / && rm -rf /libmcrypt* \
&& tar zxvf /mhash-0.9.9.9.tar.gz && cd mhash-0.9.9.9 && ./configure && make && make install && cd - / && rm -rf /mhash* \
&& tar zxvf /mcrypt-2.6.8.tar.gz && cd mcrypt-2.6.8 && LD_LIBRARY_PATH=/usr/local/lib ./configure && make && make install && cd - / && rm -rf /mcrypt*
```

### 开始安装

使用`./configure --help`查看编译支持的选项。如果写了不支持的选项，如php7里不支持`--with-mysql=mysqlnd`会提示：
``` shell
configure: WARNING: unrecognized options: --with-mysql
```

``` shell
wget http://cn2.php.net/distributions/php-7.0.7.tar.bz2
tar jxvf php-7.0.7.tar.bz2 
cd php-7.0.7

$ ./configure --prefix=/usr/local/php --with-config-file-scan-dir=/usr/local/php/etc/ --enable-inline-optimization --enable-opcache --enable-session --enable-fpm --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-pdo-sqlite --with-sqlite3 --with-gettext --enable-mbregex --enable-mbstring --enable-xml --with-iconv --with-mcrypt --with-mhash --with-openssl --enable-bcmath --enable-soap --with-xmlrpc --with-libxml-dir --enable-pcntl --enable-shmop --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-sockets --with-curl --with-curlwrappers --with-zlib --enable-zip --with-bz2 --with-gd --enable-gd-native-ttf --with-jpeg-dir --with-png-dir --with-freetype-dir --with-iconv-dir --with-readline
 
$ make
$ make install 
```

可选项：
```
--with-fpm-user=www --with-fpm-group=www
```

这里面开启了很多扩展。如果这时候忘了开启，以后还能加上吗？答案是可以的。以后只需要进入源码的`ext`目录，例如忘了`pdo_mysql`，进入`ext/pdo_mysql`，使用phpize工具，像安装普通扩展一样即可生成pdo_mysql.so。

关于：`--enable-safe-mode`
开启的话php可以执行一下系统函数，建议关闭（可搜索受此函数影响的php函数）
```
#如果只需要配置某一个目录可以执行则 设置为on并指定 safe_mode_exec_dir=string目录来执行系统函数。
#本特性已自 PHP 5.3.0 起废弃并将自 PHP 5.4.0 起移除。
safe_mode = off
```
php7编译不用加这个配置。

编译比较耗内存和CPU。等待半小时左右，编译完成：
``` shell
Build complete.
Don't forget to run 'make test'.

Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20151012/
Installing PHP CLI binary:        /usr/local/php/bin/
Installing PHP CLI man page:      /usr/local/php/php/man/man1/
Installing PHP FPM binary:        /usr/local/php/sbin/
Installing PHP FPM config:        /usr/local/php/etc/
Installing PHP FPM man page:      /usr/local/php/php/man/man8/
Installing PHP FPM status page:   /usr/local/php/php/php/fpm/
Installing phpdbg binary:         /usr/local/php/bin/
Installing phpdbg man page:       /usr/local/php/php/man/man1/
Installing PHP CGI binary:        /usr/local/php/bin/
Installing PHP CGI man page:      /usr/local/php/php/man/man1/
Installing build environment:     /usr/local/php/lib/php/build/
Installing header files:           /usr/local/php/include/php/
Installing helper programs:       /usr/local/php/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php/lib/php/
[PEAR] Archive_Tar    - installed: 1.4.0
[PEAR] Console_Getopt - installed: 1.4.1
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util       - installed: 1.3.0
[PEAR] PEAR           - installed: 1.10.1
Wrote PEAR system config file at: /usr/local/php/etc/pear.conf
You may want to add: /usr/local/php/lib/php to your php.ini include_path
/php-7.0.7/build/shtool install -c ext/phar/phar.phar /usr/local/php/bin
ln -s -f phar.phar /usr/local/php/bin/phar
Installing PDO headers:           /usr/local/php/include/php/ext/pdo/

[root@e8ed9b00e80c php-7.0.7]# /usr/local/php/bin/php -m
[PHP Modules]
bcmath
bz2
Core
ctype
curl
date
dom
fileinfo
filter
gd
gettext
hash
iconv
json
libxml
mbstring
mcrypt
mysqli
mysqlnd
openssl
pcntl
pcre
PDO
pdo_mysql
pdo_sqlite
Phar
posix
readline
Reflection
session
shmop
SimpleXML
soap
sockets
SPL
sqlite3
standard
sysvmsg
sysvsem
sysvshm
tokenizer
xml
xmlreader
xmlrpc
xmlwriter
zip
zlib

[Zend Modules]
```
 
### 配置文件
需要从安装包里复制php.ini、php-fpm.conf到安装目录：
``` shell
$ cp php-7.0.7/php.ini* /usr/local/php/etc/

$ cd /usr/local/php/etc/

$ cp php.ini-production php.ini
$ cp php-fpm.conf.default  php-fpm.conf

$ cp php-fpm.d/www.conf.default php-fpm.d/www.conf

$ ls
pear.conf  php-fpm.conf.default  php.ini-development  php.ini-production
```

**配置php.ini**
``` ini
# 不显示错误，默认
display_errors = Off

# 在关闭display_errors后开启PHP错误日志（路径在php-fpm.conf中配置），默认
log_errors = On

# 字符集，默认
default_charset = "UTF-8"

# 文件上传大小，默认 
upload_max_filesize = 2M

# 设置PHP的扩展库路径,，默认被注释了。
extension_dir = "/usr/local/php7/lib/php/extensions/no-debug-non-zts-20151012/"
# 如果不设置extension_dir，也可以直接写绝对位置：
# extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20151012/redis.so


# 设置PHP的时区
date.timezone = PRC

# 开启opcache，默认是0
[opcache]
; Determines if Zend OPCache is enabled
opcache.enable=1


```

**配置php-fpm.conf**

``` ini
; 去掉里分号，方便以后重启。建议修改
; Default Value: none
; 下面的值最终目录是/usr/local/php/var/run/php-fpm.pid
; 开启后可以平滑重启php-fpm
pid = run/php-fpm.pid

; 设置错误日志的路径，可以默认值
; Note: the default prefix is /usr/local/php/var
; Default Value: log/php-fpm.log, 即/usr/local/php/var/log/php-fpm.log
error_log = /var/log/php-fpm/error.log

; Log等级，可以默认值
; Possible Values: alert, error, warning, notice, debug
; Default Value: notice
log_level = notice

; 后台运行，默认yes，可以默认值
; Default Value: yes
;daemonize = yes

; 引入www.conf文件中的配置，可以默认值
include=/usr/local/php/etc/php-fpm.d/*.conf
```

**配置www.conf（在php-fpm.d目录下）**

www.conf这是php-fpm进程服务的扩展配置文件：

``` ini
; 设置用户和用户组，默认都是nobody。可以默认值
user = nginx
group = nginx

; 设置PHP监听
; 下面是默认值，不建议使用。可以默认值
; listen = 127.0.0.1:9000
; 根据nginx.conf中的配置fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
listen = /var/run/php-fpm/php-fpm.sock

######开启慢日志。可以默认值
slowlog = /var/log/php-fpm/$pool-slow.log
request_slowlog_timeout = 10s
```

保存配置文件后，检验配置是否正确的方法为:
``` shell
/usr/local/php/sbin/php-fpm -t
```
如果出现诸如 `test is successful` 字样，说明配置没有问题。另外该命令也可以让我们知道php-fpm的配置文件在哪。

建立软连接：
``` shell
ln -sf /usr/local/php/sbin/php-fpm /usr/bin/
ln -sf /usr/local/php/bin/php /usr/bin/
ln -sf /usr/local/php/bin/phpize /usr/bin/
ln -sf /usr/local/php/bin/php-config /usr/bin/
ln -sf /usr/local/php/bin/php-cig /usr/bin/
```

或者将php编译生成的`bin`目录添加到当前Linux系统的环境变量中:
``` shell 
echo -e '\nexport PATH=/usr/local/php/bin:/usr/local/php/sbin:$PATH\n' >> /etc/profile && source /etc/profile
```

### 启动php-fpm
``` shell
 /usr/local/php/sbin/php-fpm 
```
如果提示没有www用户（www.conf里填写了www而不是nobody），则新增：
``` shell
useradd www
chown -R www:www /www
```

检测是否启动:
``` shell
ps aux |grep php-fpm # 另外该命令也可以让我们知道fpm的配置文件在哪。
netstat -ant |grep 9000
```

查看php-fpm进程数：
``` shell
ps aux | grep -c php-fpm
```

php-fpm操作汇总：
``` shell
/usr/local/php/sbin/php-fpm 		# php-fpm启动
kill -INT `cat /usr/local/php/var/run/php-fpm.pid` 		# php-fpm关闭
kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid` 		#php-fpm平滑重启
```

重启方法二：
``` shell
killall php-fpm
/usr/local/php/sbin/php-fpm &
```

如果无法平滑启动，那就终止进程id：
``` shell
ps aux | grep php-fpm
kill -9  1210  #1210指php-fpm进程id
```

## 安装Nginx

nginx news
http://nginx.org/

http://nginx.org/download/nginx-1.11.1.tar.gz

依赖：
``` shell

# 为了支持rewrite功能，我们需要安装pcre
yum install pcre-devel

# 需要ssl的支持，如果不需要ssl支持，请跳过这一步
# yum install openssl*

# gzip 类库安装，按需安装
# yum install zlib zlib-devel
```

配置编译参数
``` shell
$ tar -zxvf nginx-1.11.1.tar.gz
$ cd nginx-1.11.1
$ ./configure \
	--prefix=/usr/local/nginx \
	--with-http_stub_status_module  \
	--with-http_ssl_module \
	--with-http_realip_module \
	--with-http_sub_module \
	--with-http_gzip_static_module \
	--with-pcre
```

配置ok：
``` shell
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + md5: using OpenSSL library
  + sha1: using OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

编译安装nginx
``` shell
make
make install
```

设置软连接：
``` shell
ln -sf /usr/local/nginx/sbin/nginx /usr/sbin 
```

检测nginx:
``` shell
nginx -t
```
显示：
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

成功了。我们重新配置下nginx.conf：

``` conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
	
	# 解决虚拟主机名字过长 http://www.jb51.net/article/26412.htm
	server_names_hash_bucket_size 128; 

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
	
	autoindex on;# 显示目录
	autoindex_exact_size on;# 显示文件大小
	autoindex_localtime on;# 显示文件时间
	
	include vhosts/*.conf;

}

```

配置localhost:
vhosts/localhost.conf
``` conf
server {
	listen       80;
	server_name  localhost;

	#charset utf-8;

	#access_log  logs/host.access.log  main;

	location / {
		root   /www/www/;
		index  index.php index.html index.htm;
	}

	#error_page  404              /404.html;

	# redirect server error pages to the static page /50x.html
	#
	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
		root   html;
	}

	# proxy the PHP scripts to Apache listening on 127.0.0.1:80
	#
	#location ~ \.php$ {
	#    proxy_pass   http://127.0.0.1;
	#}

	# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	#
	location ~ \.php$ {
		root           /www/www/;
		fastcgi_pass   127.0.0.1:9000;
		fastcgi_index  index.php;
		fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
		include        fastcgi_params;
	}
}
```

启动nginx:
``` shell
/usr/local/nginx/sbin/nginx

# 或者
nginx
```

重启：
``` shell
/usr/local/nginx/sbin/nginx -s reload

# 或者
nginx -s reload
```

停止：
``` shell
/usr/local/nginx/sbin/nginx -s stop

# 或者
nginx -s stop
```

如果提示80端口被占用了，可以使用`ps aunx | grep 80`查看。一般是apache占用了。可以使用：
``` shell
chkconfig --list
chkconfig nginx on
service apache off
```
禁止apache启动并关闭apache服务。

## 安装扩展

### 安装swoole
Swoole: PHP的异步、并行、高性能网络通信引擎
http://www.swoole.com/

``` shell
wget https://github.com/swoole/swoole-src/archive/swoole-1.8.5-stable.zip
unzip swoole-1.8.5-stable.zip
cd swoole-1.8.5-stable
phpize
./configure
make && make install
```

### 安装redis
服务器端：
http://download.redis.io/releases/redis-3.2.0.tar.gz
```
$ wget http://download.redis.io/releases/redis-3.2.0.tar.gz
$ tar xzf redis-3.2.0.tar.gz
$ cd redis-3.2.0
$ make
```
默认编译完后在当前目录的src目录下。可以复制可执行文件到其他地方：
```
mkdir /usr/local/redis
cd src
cp  redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-sentinel redis-server redis-trib.rb /usr/local/redis
```

复制配置文件
```
$ cd redis-3.2.0
$ cp redis.conf /usr/local/redis/
```

或者安装的时候指定位置：
```
make PREFIX=/usr/local/redis install
```

**将Redis的命令所在目录添加到系统参数PATH中:**
修改profile文件：
```
vi /etc/profile
```
在最后行追加: 
```
export PATH="$PATH:/usr/local/redis/bin"
```
然后马上应用这个文件： 
```
. /etc/profile  
```
这样就可以直接调用redis-cli的命令了

客户端：
#### 2.0安装
```
wget https://github.com/nicolasff/phpredis/archive/2.2.4.tar.gz
tar -zxvf 2.2.4
cd phpredis-2.2.4/
phpize
./configure 
make && make install
```

#### 3.0安装
phpredis/phpredis: A PHP extension for Redis
https://github.com/phpredis/phpredis

需要先安装igbinary：

PECL :: Package :: igbinary
http://pecl.php.net/package/igbinary

```
wget http://pecl.php.net/get/igbinary-1.2.1.tgz
tar zxvf igbinary-1.2.1.tgz
cd igbinary-1.2.1
phpize
./configure 
make && make install
```

```
wget https://github.com/phpredis/phpredis/archive/3.0.0-rc1.zip
unzip 3.0.0-rc1
cd phpredis-3.0.0-rc1/

phpize
./configure [--enable-redis-igbinary]
make && make install
```

### 安装memcache

## 使用yum安装php

如果只是想快速搭建开发环境，可以使用yum进行安装：
```
yum install php php-gd php-fpm php-mbstring php-mcrypt php-mysql php-sqlite3 php-pdo php-memcache php-devel
```
使用`yum search php`可以查询到可以安装哪些php相关的扩展。

需要注意的是，使用yum安装的php,其配置文件位于：
```
/etc/php.ini
/etc/php-fpm.conf
/etc/php-fpm.d/www.conf

/etc/php.d/mysqli.ini
/etc/php.d/pdo.ini
```
ubuntu与centos不一样，会分开存放php.ini到`/etc/php7/cli/php.ini`和`/etc/php7/fpm/php.ini`里。这个需要注意。

yum无法安装的扩展，可以使用pecl安装。由于pecl是需要编译的，所以需要先安装编译器：
```
yum install -y gcc gcc-c++ make cmake bison autoconf
```

然后：
```
pecl install swoole
pecl install redis
```

pecl安装扩展完成后会提示添加so文件到php.ini。示例：
```
Build process completed successfully
Installing '/usr/lib64/php/modules/swoole.so'
install ok: channel://pecl.php.net/swoole-1.9.11
configuration option "php_ini" is not set to php.ini location
You should add "extension=swoole.so" to php.ini
```
添加示例：
```
[swoole]
extension = /usr/lib64/php/modules/swoole.so
```

使用`php -m`可以查看安装的扩展。

## 信号管理
不重载配置启动新/旧工作进程
```
kill -HUP 旧/新版主进程号
```

从容关闭旧/新进程
```
kill -QUIT 旧/新主进程号
```

如果此时报错，提示还有进程没有结束就用下面命令先关闭旧/新工作进程，再关闭主进程号：
```
kill -TERM 旧/新工作进程号
```

升级、添加或删除模块时，我们需要停掉服务器
```
kill -USR2 旧版程序的主进程号或进程文件名
```

## MySQL
如果存在旧版本，需要先卸载旧版本。

使用下面的命令检查是否安装有MySQL Server
```
rpm -qa | grep mysql
```
有的话通过下面的命令来卸载掉
```
rpm -e mysql   //普通删除模式
rpm -e --nodeps mysql    // 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
```

### 安装

安装编译代码需要的包
```
yum -y install make gcc-c++ cmake bison-devel  ncurses-devel
```

下载MySQL 5.6.14：
```
wget http://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.14.tar.gz
tar xvf mysql-5.6.14.tar.gz
cd mysql-5.6.14
```

编译安装
```
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci\
-DMYSQL_USER=mysql

make && make install
```

编译的参数可以参考http://dev.mysql.com/doc/refman/5.5/en/source-configuration-options.html。

整个过程需要30分钟左右……漫长的等待

### 配置MySQL

#### 设置权限

使用下面的命令查看是否有mysql用户及用户组
```
cat /etc/passwd 查看用户列表
cat /etc/group  查看用户组列表
```
如果没有就创建
```
groupadd mysql
useradd -g mysql mysql
```
修改`/usr/local/mysql`权限
```
chown -R mysql:mysql /usr/local/mysql
```
修改`/usr/local/mysql`权限

#### 初始化配置

进入安装路径
```
cd /usr/local/mysql
```
进入安装路径，执行初始化配置脚本，创建系统自带的数据库和表
```
scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
```
注：在启动MySQL服务时，会按照一定次序搜索my.cnf，先在/etc目录下找，找不到则会搜索"$basedir/my.cnf"，在本例中就是 /usr/local/mysql/my.cnf，这是新版MySQL的配置文件的默认位置！

注意：在CentOS 6.4版操作系统的最小安装完成后，在/etc目录下会存在一个my.cnf，需要将此文件更名为其他的名字，如：/etc/my.cnf.bak，否则，该文件会干扰源码安装的MySQL的正确配置，造成无法启动。

在使用"yum update"更新系统后，需要检查下/etc目录下是否会多出一个my.cnf，如果多出，将它重命名成别的。否则，MySQL将使用这个配置文件启动，可能造成无法正常启动等问题。

### 启动MySQL

添加服务，拷贝服务脚本到init.d目录，并设置开机启动
```
cp support-files/mysql.server /etc/init.d/mysql
chkconfig mysql on
service mysql start  --启动MySQL
```

### 配置用户

MySQL启动成功后，root默认没有密码，我们需要设置root密码。

设置之前，我们需要先设置PATH，要不不能直接调用mysql

修改/etc/profile文件，在文件末尾添加
```
PATH=/usr/local/mysql/bin:$PATH
export PATH
```
关闭文件，运行下面的命令，让配置立即生效
```
source /etc/profile
```
现在，我们可以在终端内直接输入mysql进入，mysql的环境了

执行下面的命令修改root密码
```
mysql -uroot  
mysql> SET PASSWORD = PASSWORD('123456');
```
若要设置root用户可以远程访问，执行
```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.16.%' IDENTIFIED BY 'password' WITH GRANT OPTION;
```
红色的password为远程访问时，root用户的密码，可以和本地不同。

### 配置防火墙

防火墙的3306端口默认没有开启，若要远程访问，需要开启这个端口

打开`/etc/sysconfig/iptables`

在“-A INPUT –m state --state NEW –m tcp –p –dport 22 –j ACCEPT”，下添加：
```
-A INPUT -m state --state NEW -m tcp -p -dport 3306 -j ACCEPT
```
然后保存，并关闭该文件，在终端内运行下面的命令，刷新防火墙配置：
```
service iptables restart
```
OK，一切配置完毕，你可以访问你的MySQL了~

注意：

CentOS 7中默认使用Firewalld做防火墙，所以修改iptables后，在重启系统后，根本不管用。

Firewalld中添加端口方法如下：
```
firewall-cmd --zone=public --add-port=3306/tcp --permanent

firewall-cmd --reload
```

## 更多资料
1、linux下为已经编译好的php环境添加mysql扩展
https://ask.hellobi.com/blog/liangyong/69
 
## 附件
相关源码包
链接: http://pan.baidu.com/s/1kUQTlFD 密码: ajzk
