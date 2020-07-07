# 3. PHP编译安装

## 安装

> tar -zxf php-7.2.15.tar.gz cd php-7.2.15

```text
./configure --prefix=/usr/local/webserver/php-7.2.15 \
--with-config-file-path=/usr/local/webserver/php-7.2.15/etc \
--with-config-file-scan-dir=/usr/local/webserver/php-7.2.15/etc/php.d \
--with-mysqli --with-pdo-mysql \
--with-openssl --with-curl=/usr/local/curl-7.54.1 --with-zlib --enable-zip --with-bz2 \
--with-iconv-dir=/usr/local --with-xpm-dir=/usr/ --disable-rpath --enable-pcntl \
--with-libxml-dir=/usr --enable-bcmath --enable-shmop --enable-sysvsem \
--with-gettext --with-mcrypt --with-mhash --enable-mbstring \
--with-gd --enable-gd-native-ttf --with-freetype-dir --with-jpeg-dir --with-png-dir \
--enable-sockets --enable-soap --with-libdir=lib64 --with-xmlrpc \
--enable-fpm --with-fpm-user=www --with-fpm-group=www
```

> make ZEND\_EXTRA\_LIBS='-liconv' make install

### 复制 PHP 服务管理文件

> cp sapi/fpm/init.d.php-fpm /usr/local/webserver/php-7.2.15/sbin/php-fpm.init
>
> cp -a /usr/local/webserver/php-7.2.15/sbin/php-fpm.init /usr/local/webserver/php-7.2.15/sbin/php-fpm.ori
>
> chmod 750 /usr/local/webserver/php-7.2.15/sbin/php-fpm.init
>
> ln -sv /usr/local/webserver/php-7.2.15/sbin/php-fpm.init /etc/init.d/php-fpm7215

### 自定义日志存储位置

> mkdir -p /media/raid10/logs/php
>
> touch /media/raid10/logs/php/error\_7215.log /media/raid10/logs/php/slow\_7215.log

### 修改 PHP 服务管理文件\(更改自定义日志存储位置的文件权限\)

> sed -i '69 a chmod 644 /media/raid10/logs/php/error\_7215.log' /etc/init.d/php-fpm7215
>
> sed -i '70 a chmod 644 /media/raid10/logs/php/slow\_7215.log' /etc/init.d/php-fpm7215
>
> sed -i '145 a chmod 644 /media/raid10/logs/php/error\_7215.log' /etc/init.d/php-fpm7215
>
> sed -i '146 a chmod 644 /media/raid10/logs/php/slow\_7215.log' /etc/init.d/php-fpm7215

### 复制 PHP 默认的服务配置文件

> cp -a php.ini-production /usr/local/webserver/php-7.2.15/etc/php.ini
>
> cp -a /usr/local/webserver/php-7.2.15/etc/php-fpm.conf.default /usr/local/webserver/php-7.2.15/etc/php-fpm.conf
>
> cp -a /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf.default /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf

### 自定义 PHP 服务配置文件内容

> sed -i 's\#include=/usr/local/webserver/php-7.2.15/etc/php-fpm.d/\*.conf\#include=/usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.conf
>
> sed -i 's\#\[www\]\#\[www-7.2.15\]\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf
>
> sed -i 's\#listen = 127.0.0.1:9000\#listen = 127.0.0.1:7215\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf
>
> sed -i 's\#pm = dynamic\#pm = static\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf sed -i 's\#pm.max\_children = 5\#pm.max\_children = 10\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf sed -i 's\#;slowlog = log/$pool.log.slow\#slowlog = /usr/local/webserver/php-7.2.15/var/log/slow\_7115.log\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf sed -i 's\#;request\_slowlog\_timeout = 0\#request\_slowlog\_timeout = 2\#' /usr/local/webserver/php-7.2.15/etc/php-fpm.d/www.conf

### 开机启动

> cp -a /etc/rc.local /etc/rc.local.before.php-7.2.15
>
> echo "" &gt;&gt; /etc/rc.local
>
> echo "\# Start PHP 7.2.15 Service" &gt;&gt; /etc/rc.local echo "\#/etc/init.d/php-fpm7215 start" &gt;&gt; /etc/rc.local

### 删除 PHP-7.2.15 安装文件

> cd /usr/local/src/ rm -rf php-7.2.15

### 测试启动和关闭 PHP-7.2.15 服务

> /etc/init.d/php-fpm7215 start
>
> netstat -tnlp \| grep 7215
>
> /etc/init.d/php-fpm7215 stop

## 安装 PHP redis 扩展

> cd /usr/local/src/
>
> tar -zxf phpredis-php7.tar.gz
>
> cd phpredis-php7
>
> /usr/local/webserver/php-7.2.15/bin/phpize
>
> ./configure --with-php-config=/usr/local/webserver/php-7.2.15/bin/php-config
>
> make
>
> make install
>
> cp -a /usr/local/webserver/php-7.2.15/etc/php.ini /usr/local/webserver/php-7.2.15/etc/php.ini.before.redis
>
> sed -i "/;extension=xsl/ a\extension=/usr/local/webserver/php-7.2.15/lib/php/extensions/no-debug-non-zts-20170718/redis.so" /usr/local/webserver/php-7.2.15/etc/php.ini
>
> cd /usr/local/src/
>
> rm -rf phpredis-php7

## 安装 PHP 老的mysql 扩展

> cd /usr/local/src/
>
> tar -zxf php7\_mysql.tar.gz
>
> cd php7\_mysql
>
> /usr/local/webserver/php-7.2.15/bin/phpize
>
> ./configure --with-php-config=/usr/local/webserver/php-7.2.15/bin/php-config
>
> make
>
> make install
>
> cp -a /usr/local/webserver/php-7.2.15/etc/php.ini /usr/local/webserver/php-7.2.15/etc/php.ini.before.mysql
>
> sed -i "/;extension=xsl/ a\extension=/usr/local/webserver/php-7.2.15/lib/php/extensions/no-debug-non-zts-20170718/mysql.so" /usr/local/webserver/php-7.2.15/etc/php.ini
>
> cd /usr/local/src/
>
> rm -rf php7\_mysql

## 安装 PHP swoole 扩展 \[ 下载路径 [http://pecl.php.net/package/swoole](http://pecl.php.net/package/swoole) \]

> cd /usr/local/src/
>
> tar -zxf swoole-4.2.13.tgz cd swoole-4.2.13
>
> /usr/local/webserver/php-7.2.15/bin/phpize
>
> ./configure --with-php-config=/usr/local/webserver/php-7.2.15/bin/php-config
>
> make
>
> make install
>
> cp -a /usr/local/webserver/php-7.2.15/etc/php.ini /usr/local/webserver/php-7.2.15/etc/php.ini.before.swoole
>
> sed -i "/;extension=xsl/ a\extension=/usr/local/webserver/php-7.2.15/lib/php/extensions/no-debug-non-zts-20170718/swoole.so" /usr/local/webserver/php-7.2.15/etc/php.ini
>
> cd /usr/local/src/
>
> rm -rf swoole-4.2.13

## 检查 redis mysql swoole 扩展

> /usr/local/webserver/php-7.2.15/bin/php -m \| grep redis /usr/local/webserver/php-7.2.15/bin/php -m \| grep mysql /usr/local/webserver/php-7.2.15/bin/php -m \| grep swoole

or

* /usr/local/webserver/php-7.2.15/bin/php --ri redis
* /usr/local/webserver/php-7.2.15/bin/php --ri mysql
* /usr/local/webserver/php-7.2.15/bin/php --ri swoole

## 开启 opcache 缓存

* sed -i "/\[opcache\]/ a\zend\_extension = opcache.so" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.enable=1\#opcache.enable=1\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.memory\_consumption=128\#opcache.memory\_consumption=256\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.interned\_strings\_buffer=8\#opcache.interned\_strings\_buffer=8\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.max\_accelerated\_files=10000\#opcache.max\_accelerated\_files=10000\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.max\_wasted\_percentage=5\#opcache.max\_wasted\_percentage=5\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.use\_cwd=1\#opcache.use\_cwd=0\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.validate\_timestamps=1\#opcache.validate\_timestamps=1\#" /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.revalidate\_freq=2\#opcache.revalidate\_freq=60\#"                     /usr/local/webserver/php-7.2.15/etc/php.ini
* sed -i "s\#;opcache.save\_comments=1\#opcache.save\_comments=0\#"                          /usr/local/webserver/php-7.2.15/etc/php.ini

### 检查 opcache 缓存

> /usr/local/webserver/php-7.2.15/bin/php -m \| grep -i opcache or

* /usr/local/webserver/php-7.2.15/bin/php --ri opcache

