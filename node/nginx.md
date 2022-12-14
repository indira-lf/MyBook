# 第1章 nginx的使用

## nginx的目录结构

- client_body_temp
- conf
- - fastcgi.conf
  - fastcgi.conf.default
  - fastcgi_params
  - fastcgi_params.default
  - koi-utf
  - koi-win
  - mime.types
  - mime.types.default
  - nginx.conf
  - nginx.conf.default
  - scgi_params
  - scgi_params.default
  - uwsgi_params
  - uwsgi_params.default
  - win-utf
- fastcgi_temp
- html
- logs
- proxy_temp
- sbin
- - nginx
- scgi_temp
- uwsgi_temp

## nginx配置与应用场景

nginx.conf配置文件

```coffeescript

#user  nobody;
#默认为1，表示开启一个业务进程 
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    #单个业务进程可接受连接数
    worker_connections  1024;
}


http {
    #引入http mime类型
    include       mime.types;
    #如果mime类型每匹配上，默认使用二进制流的方式传输
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    #使用linux的sendfile(socket,file,len)高效网络传输，也就是数据0拷贝
    sendfile        on; 
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80; #监听端口号
        server_name  localhost; #主机名

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / { #匹配路径
            root   html; #文件根目录
            index  index.html index.htm; #默认页名称
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #错误码500 502 503 504返回50x.html页面
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
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


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

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

### sendfile配置

未开启时，需要拷贝文件

![sendfile配置no](assets/sendfile配置no.png)

开启后

![sendfile配置yes](assets/sendfile配置yes.png)

### server_name配置规则

> 匹配顺序：
>
> 1. 精确的名字
> 2. 以*号开头的最长通配符名称，例如 *.example.org
> 3. 以*号结尾的最长通配符名称，例如 mail. *
> 4. 第一个匹配的正则表达式（在配置文件中出现的顺序）

**1、修改本地hosts文件** 

```properties
47.103.96.17 llf.nginxa.com
47.103.96.17 www.nginxa.com
```

**服务器创建index.html**

```
mkdir /www/www
mkdir /www/vod

vim /www/www/index.html
vim /www/vod/index.html
```

**修改nginx.conf配置**

1. 不同端口

```coffeescript
    server {
        listen       81;
        server_name  nginxa.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /www/www;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    server {
        listen       82;
        server_name  llf.nginxa.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /www/vod;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

2. 相同端口

```coffeescript
1、完整匹配
	server_name  llf.nginxa.com;

2、通配符匹配
	server_name  *.nginxa.com;

3、通配符结束匹配
	server_name  llf.*;

4、正则匹配
	server_name ~^[0-9]+\.nginxa\.com$;
```



## 反向代理

**概念** 

`反向代理（Reverse Proxy）` 方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

**配置修改** 

```coffeescript
    server {
        listen       80;
        server_name  www.atnginx.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   /www/www;
            #index  index.html index.htm;
			
			proxy_pass http://www.atguigu.com;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```
## 负载均衡

###  负载均衡策略

**1、轮询** 

```coffeescript
	upstream httpds {
		server 192.168.44.102:80;
		server 192.168.44.103:80;
	}

    location / {
        #root   /www/www;
        #index  index.html index.htm;
		
		proxy_pass http://httpds; # httpds为别名
    }
```
**2、权重** 

```coffeescript
upstream httpds {
    # weight 加负载均衡的权重，down、backup不常用
	server 192.168.44.102:80 weight=8 down; #加down后不参加负载均衡
	server 192.168.44.103:80 weight=2;
	server 192.168.44.104:80 weight=1 backup; #backup 备用机
}

location / {
    #root   /www/www;
    #index  index.html index.htm;
	
	proxy_pass http://httpds; # httpds为别名
}
```
**3、ip_hash** 

根据客户端的ip地址转发同一台服务器，可以保持会话。

**4、least_conn** 

最少连接访问

**5、url_hash**

根据用户访问的url定向转发请求

**6、fair** 

根据后端服务器响应时间转发请求