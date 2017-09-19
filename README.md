# Linux 编译安装 Nginx

Nginx的安装依赖于以下三个包，意思就是在安装Nginx之前首先必须安装一下的三个包，安装顺序为以下的顺序：

1、SSL功能需要openssl库，下载地址：http://www.openssl.org/

2、gzip模块需要zlib库，下载地址：http://www.zlib.net/

3、rewrite模块需要pcre库，下载地址：http://www.pcre.org/

4、Nginx的安装包：下载地址为：http://nginx.org/en/download.html

**_CentOS系统_ 安装编译工具及库文件：**

> yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel

**命令下载：**

wget http://www.zlib.net/zlib-1.2.11.tar.gz

wget https://www.openssl.org/source/openssl-1.0.2l.tar.gz

wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz

wget http://nginx.org/download/nginx-1.12.1.tar.gz

程序一般安装在 `/usr/local/` 目录下

**安装pcre：**

1、解压pcre：

> tar -zxvf pcre-8.40.tar.gz

2、进入安装包目录：

> cd pcre-8.40

3、配置、编译、安装：

> ./configure

> make

> make install

**安装Nginx：**

1、解压安装包：

tar -zxvf nginx-1.12.1.tar.gz

2、进入安装包目录：

> cd nginx-1.12.1

3、配置、编译、安装：

> ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/pcre-8.40

> make

> make install

    注意：--with-pcre=DIR 是pcre源码目录，而不是编译安装后的目录。

4、查看Nginx版本：

> /usr/local/nginx/sbin/nginx -v

**创建 Nginx 运行使用的用户 www：**

> /usr/sbin/groupadd www

> /usr/sbin/useradd -g www www

**配置 `nginx.conf` ：**

**参考以下：**
```
user www;   #设置权限用户
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

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile             on;
    tcp_nopush           on;
    tcp_nodelay          on;
    client_max_body_size 20M;

    #keepalive_timeout  0;
    keepalive_timeout  2;
    types_hash_max_size 2048;

    open_file_cache          max=65535  inactive=20s;
    open_file_cache_valid    30s;
    open_file_cache_min_uses 5;
    open_file_cache_errors   off;

    gzip on;
    gzip_disable "msie6";
    gzip_min_length 1k;
    gzip_http_version 1.1;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_proxied any;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml application/json image/jpeg image/jpg image/png image/gif;
    gzip_buffers 8 64k;

    upstream php_cgi {
        server 127.0.0.1:9000;
    }

    server {
        listen 80;
        server_name baidu.com www.baidu.com;

        root          /var/www/www.baidu.com;   #项目路径
		index         index.html index.htm;

		location ~* \.(jpg|jpeg|gif|css|png|js|ico)$ {
			access_log off;
			expires max;
		}

		if (!-d $request_filename) {
			rewrite ^/(.+)/$ /$1 permanent;
		}

		location / {
			try_files $uri $uri/ /index.php?$query_string;
		}

		location ~ \.php$ {
			try_files   $uri /index.php =404;
			fastcgi_split_path_info ^(.+\.php)(/.+)$;
			fastcgi_pass    php_cgi;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
			include fastcgi_params;
		}
    }

    #http:// 重定向 https://
    server {
        listen 80;
        server_name baidu.com www.baidu.com;

        rewrite ^/(.*)$ https://www.baidu.com/$1 permanent;
    }

    include sites/*.conf;

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #证书路径
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;
	#    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	#定义算法
	#    ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS !RC4";

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
}
```

**检查conf配置是否正确：**

> /usr/local/nginx/sbin/nginx -t

**启动Nginx：**

> /usr/local/nginx/sbin/nginx

**重新载入Nginx配置（平滑重启）：**

> /usr/local/nginx/sbin/nginx -s reload

**重启Nginx：**

> /usr/local/nginx/sbin/nginx

**快速关闭Nginx：**

> /usr/local/nginx/sbin/nginx -s stop

**正常关闭Nginx：**

> /usr/local/nginx/sbin/nginx -s quit

**重新打开日志文件：**

> /usr/local/nginx/sbin/nginx -s reopen

