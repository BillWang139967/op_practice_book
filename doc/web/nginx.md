# Nginx 

* [安装](#安装)
* [Nginx 配置文件详解](#nginx-配置文件详解)
* [Nginx服务器基础配置指令](#nginx服务器基础配置指令)
	* [nginx.conf文件的结构](#nginxconf文件的结构)
	* [配置运行Nginx服务器用户（组）](#配置运行nginx服务器用户组)
	* [配置允许生成的worker process数](#配置允许生成的worker-process数)
	* [配置Nginx进程PID存放路径](#配置nginx进程pid存放路径)
	* [配置错误日志的存放路径](#配置错误日志的存放路径)
	* [配置文件的引入](#配置文件的引入)
	* [设置网络连接的序列化](#设置网络连接的序列化)
	* [设置是否允许同时接收多个网络连接](#设置是否允许同时接收多个网络连接)
	* [事件驱动模型的选择](#事件驱动模型的选择)
	* [配置最大连接数](#配置最大连接数)
	* [定义MIME-Type](#定义mime-type)
	* [自定义服务日志](#自定义服务日志)
	* [配置允许sendfile方式传输文件](#配置允许sendfile方式传输文件)
	* [配置连接超时时间](#配置连接超时时间)
	* [单连接请求数上限](#单连接请求数上限)
	* [配置网络监听](#配置网络监听)
	* [**基于名称的虚拟主机配置**](#基于名称的虚拟主机配置)
	* [**基于IP的虚拟主机配置**](#基于ip的虚拟主机配置)
	* [**配置location块**(重中之重)](#配置location块重中之重)
	* [配置请求的根目录](#配置请求的根目录)
	* [更改location的URI](#更改location的uri)
	* [设置网站的默认首页](#设置网站的默认首页)
	* [设置网站的错误页面](#设置网站的错误页面)
	* [基于IP配置Nginx的访问权限](#基于ip配置nginx的访问权限)
	* [基于密码配置Nginx的访问权限](#基于密码配置nginx的访问权限)
* [Nginx服务器基础配置实例](#nginx服务器基础配置实例)
	* [测试myServer1的访问](#测试myserver1的访问)
	* [测试myServer2的访问](#测试myserver2的访问)
* [Nginx服务器架构](#nginx服务器架构)
	* [模块化结构](#模块化结构)
		* [什么是模块化开发？](#什么是模块化开发)
		* [Nginx的模块化结构](#nginx的模块化结构)
	* [Nginx的模块清单](#nginx的模块清单)
	* [Nginx的web请求处理机制](#nginx的web请求处理机制)
	* [Nginx的事件驱动模型](#nginx的事件驱动模型)
* [应用](#应用)
	* [架设简单文件服务器](#架设简单文件服务器)


# 安装

安装 Nginx 之前，确保系统已经安装 gcc、openssl-devel、pcre-devel 和 zlib-devel软件库

* gcc、openssl-devel、zlib-devel可以通过光盘直接选择安装
* pcre-devel 安装pcre库是为了使Nginx支持HTTP Rewrite模块

```
#wget http://nginx.org/download/nginx-1.8.0.tar.gz 
#tar xzvf nginx-1.8.0.tar.gz
#cd nginx-1.8.0
# ./configure --prefix=/opt/X_nginx/nginx
#make && sudo make install
```
# Nginx 配置文件详解

```
#定义Nginx运行的用户和用户组
user www www;

#nginx进程数，建议设置为等于CPU总核心数。
worker_processes 8;

#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log /var/log/nginx/error.log info;

#进程文件
pid /var/run/nginx.pid;

#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除，但是nginx分配请求并不均匀，所以建议与ulimit -n的值保持一致。
worker_rlimit_nofile 65535;

#工作模式与连接数上限
events
{
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
    use epoll;
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections 65535;
}

#设定http服务器
http {
    include mime.types; #文件扩展名与文件类型映射表
    default_type application/octet-stream; #默认文件类型
    #charset utf-8; #默认编码
    server_names_hash_bucket_size 128; #服务器名字的hash表大小
    client_header_buffer_size 32k; #上传文件大小限制
    large_client_header_buffers 4 64k; #设定请求缓
    client_max_body_size 8m; #设定请求缓
    sendfile on; #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    autoindex on; #开启目录列表访问，合适下载服务器，默认关闭。
    tcp_nopush on; #防止网络阻塞
    tcp_nodelay on; #防止网络阻塞
    keepalive_timeout 120; #长连接超时时间，单位是秒

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers 4 16k; #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2; #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;
    #limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用

    upstream blog.ha97.com {
        #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;
    }

    #虚拟主机的配置
    server {
        #监听端口
        listen 80;
        #域名可以有多个，用空格隔开
        server_name www.ha97.com ha97.com;
        index index.html index.htm index.php;
        root /data/www/ha97;
        location ~ .*.(php|php5)?$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }
    
        #图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires 10d;
        }
        
        #JS和CSS缓存时间设置
        location ~ .*.(js|css)?$ {
            expires 1h;
        }

        #日志格式设定
        log_format access '$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" $http_x_forwarded_for';

        #定义本虚拟主机的访问日志
        access_log /var/log/nginx/ha97access.log access;

        #对 "/" 启用反向代理
        location / {
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;
            client_max_body_size 10m; #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
            proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的设置
            proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_path /dev/shm/proxy_temp; #类似于http核心模块中的client_body_temp_path指令，指定一个目录来缓冲比较大的被代理请求。 
             
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file conf/htpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
        }

        #本地动静分离反向代理配置
        #所有jsp的页面均交由tomcat或resin处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }

        #所有静态文件由nginx直接读取不经过tomcat或resin
        location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)$
        { expires 15d; }

        location ~ .*.(js|css)?$
        { expires 1h; }
    }
}
```

# Nginx服务器基础配置指令

## nginx.conf文件的结构

+ Global: nginx运行相关
+ events: 与用户的网络连接相关
+ http
    + http Global: 代理，缓存，日志，以及第三方模块的配置
    + server
        + server Global: 虚拟主机相关
        + location: 地址定向，数据缓存，应答控制，以及第三方模块的配置

> 所有的所有的所有的指令，都要以`;`结尾

## 配置运行Nginx服务器用户（组）

user nobody nobody;

## 配置允许生成的worker process数

worker_processes auto;
worker_processes 4;

> 这个数字，跟电脑CPU核数要保持一致

```
# grep ^proces /proc/cpuinfo
processor       : 0
processor       : 1
processor       : 2
processor       : 3
# grep ^proces /proc/cpuinfo | wc -l
4
```

## 配置Nginx进程PID存放路径

pid logs/nginx.pid;

> 这里面保存的就是一个数字，nginx master 进程的进程号

## 配置错误日志的存放路径

error_log logs/error.log;
error_log logs/error.log error;

## 配置文件的引入

include mime.types;
include fastcgi_params;
include ../../conf/*.conf;

## 设置网络连接的序列化

accept_mutex on;

> 对多个nginx进程接收连接进行序列化，防止多个进程对连接的争抢（惊群）

## 设置是否允许同时接收多个网络连接

multi_accept off;

## 事件驱动模型的选择

use select|poll|kqueue|epoll|rtsig|/dev/poll|eventport

> 这个重点，后面再看

## 配置最大连接数

worker_connections 512;

## 定义MIME-Type

include mime.types;
default_type application/octet-stream;

## 自定义服务日志

access_log logs/access.log main;
access_log off;

## 配置允许sendfile方式传输文件

sendfile off;

sendfile on;
sendfile_max_chunk 128k;

> nginx 每个worker process 每次调用 sendfile()传输的数据量的最大值

Refer:

+ [Linux kenel sendfile 如何提升性能](http://www.vpsee.com/2009/07/linux-sendfile-improve-performance/)
+ [nginx sendifle tcp_nopush tcp_nodelay参数解释](http://blog.csdn.net/zmj_88888888/article/details/9169227)

## 配置连接超时时间

> 与用户建立连接后，Nginx可以保持这些连接一段时间, 默认 75s
> 下面的65s可以被Mozilla/Konqueror识别，是发给用户端的头部信息`Keep-Alive`值

keepalive_timeout 75s 65s;

## 单连接请求数上限

> 和用户端建立连接后，用户通过此连接发送请求;这条指令用于设置请求的上限数

keepalive_requests 100;

## 配置网络监听

listen *:80 | *:8000; # 监听所有的80和8000端口

listen 192.168.1.10:8000;
listen 192.168.1.10;
listen 8000; # 等同于 listen *:8000;
listen 192.168.1.10 default_server backlog=511; # 该ip的连接请求默认由此虚拟主机处理;最多允许1024个网络连接同时处于挂起状态

## **基于名称的虚拟主机配置**

server_name myserver.com www.myserver.com;

server_name *.myserver.com www.myserver.* myserver2.*; # 使用通配符

> 不允许的情况： server_name www.ab*d.com; # `*`只允许出现在www和com的位置

server_name ~^www\d+\.myserver\.com$; # 使用正则

> nginx的配置中，可以用正则的地方，都以`~`开头

> from Nginx~0.7.40 开始，server_name 中的正则支持 字符串捕获功能（capture）

server_name ~^www\.(.+)\.com$; # 当请求通过www.myserver.com请求时， myserver就被记录到`$1`中, 在本server的上下文中就可以使用

如果一个名称 被多个虚拟主机的 server_name 匹配成功, 那这个请求到底交给谁处理呢？看优先级：

1. 准确匹配到server_name
2. 通配符在开始时匹配到server_name
3. 通配符在结尾时匹配到server_name
4. 正则表达式匹配server_name
5. 先到先得

## **基于IP的虚拟主机配置**

> 基于IP的虚拟主机，需要将网卡设置为同时能够监听多个IP地址

```
ifconfig
# 查看到本机IP地址为 192.168.1.30
ifconfig eth1:0 192.168.1.31 netmask 255.255.255.0 up
ifconfig eth1:1 192.168.1.32 netmask 255.255.255.0 up
ifconfig
# 这时就看到eth1增加来2个别名， eth1:0 eth1:1

# 如果需要机器重启后仍保持这两个虚拟的IP
echo "ifconfig eth1:0 192.168.1.31 netmask 255.255.255.0 up" >> /etc/rc.local
echo "ifconfig eth1:0 192.168.1.32 netmask 255.255.255.0 up" >> /etc/rc.local
```

再来配置基于IP的虚拟主机

```
http {
    ...
    server {
     listen 80;
     server_name 192.168.1.31;
     ...
    }
    server {
     listen 80;
     server_name 192.168.1.32;
     ...
    }
}
```

## **配置location块**(重中之重)

> location 块的配置，应该是最常用的了

location [ = | ~ | ~* | ^~ ] uri {...}

这里内容分2块，匹配方式和uri， 其中uri又分为 标准uri和正则uri

先不考虑 那4种匹配方式

1. Nginx首先会再server块的多个location中搜索是否有`标准uri`和请求字符串匹配， 如果有，记录匹配度最高的一个；
2. 然后，再用location块中的`正则uri`和请求字符串匹配， 当第一个`正则uri`匹配成功，即停止搜索， 并使用该location块处理请求；
3. 如果，所有的`正则uri`都匹配失败，就使用刚记录下的匹配度最高的一个`标准uri`处理请求
4. 如果都失败了，那就失败喽

再看4种匹配方式：

+ `=`:  用于`标准uri`前，要求请求字符串与其严格匹配，成功则立即处理
+ `^~`: 用于`标准uri`前，并要求一旦匹配到，立即处理，不再去匹配其他的那些个`正则uri`
+ `~`:  用于`正则uri`前，表示uri包含正则表达式， 并区分大小写
+ `~*`: 用于`正则uri`前， 表示uri包含正则表达式， 不区分大小写

> `^~` 也是支持浏览器编码过的URI的匹配的哦， 如 `/html/%20/data` 可以成功匹配 `/html/ /data`

## 配置请求的根目录

Web服务器收到请求后，首先要在服务端指定的目录中寻找请求资源

root /var/www;

## 更改location的URI

除了使用root指明处理请求的根目录，还可以使用alias 改变location收到的URI的请求路径

```
location ~ ^/data/(.+\.(htm|html))$ {
    alias /locatinotest1/other/$1;
}
```

## 设置网站的默认首页

index 指令主要有2个作用：

+ 对请求地址没有指明首页的，指定默认首页
+ 对一个请求，根据请求内容而设置不同的首页，如下：

```
location ~ ^/data/(.+)/web/$ {
    index index.$1.html index.htm;
}
```
## 设置网站的错误页面

error_page 404 /404.html;
error_page 403 /forbidden.html;
error_page 404 =301 /404.html;

```
location /404.html {
    root /myserver/errorpages/;
}
```

## 基于IP配置Nginx的访问权限

```
location / {
    deny 192.168.1.1;
    allow 192.168.1.0/24;
    allow 192.168.1.2/24;
    deny all;
}
```
> 从192.168.1.0的用户时可以访问的，因为解析到allow那一行之后就停止解析了


## 基于密码配置Nginx的访问权限

auth_basic "please login";
auth_basic_user_file /etc/nginx/conf/pass_file;

> 这里的file 必须使用绝对路径，使用相对路径无效

```
# /usr/local/apache2/bin/htpasswd -c -d pass_file user_name
# 回车输入密码，-c 表示生成文件，-d 是以 crypt 加密。

name1:password1
name2:password2:comment
```

> 经过basic auth认证之后没有过期时间，直到该页面关闭；
> 如果需要更多的控制，可以使用 HttpAuthDigestModule http://wiki.nginx.org/HttpAuthDigestModule

# Nginx服务器基础配置实例

```
user nginx nginx;

worker_processes 3;

error_log logs/error.log;
pid myweb/nginx.pid;

events {
    use epoll;
    worker_connections 1024;
}

http {
    include mime.types;
    default_type applicatioin/octet-stream;

    sendfile on;

    keepalive_timeout 65;

    log_format access.log '$remote_addr [$time_local] "$request" "$http_user_agent"';

    server {
        listen 8081;
        server_name myServer1;

        access_log myweb/server1/log/access.log;
        error_page 404 /404.html;

        location /server1/location1 {
            root myweb;
            index index.svr1-loc1.htm;
        }

        location /server1/location2 {
            root myweb;
            index index.svr1-loc2.htm;
        }
    }

    server {
        listen 8082;
        server_name 192.168.0.254;

        auth_basic "please Login:";
        auth_basic_user_file /opt/X_nginx/Nginx/myweb/user_passwd;

        access_log myweb/server2/log/access.log;
        error_page 404 /404.html;

        location /server2/location1 {
            root myweb;
            index index.svr2-loc1.htm;
        }

        location /svr2/loc2 {
            alias myweb/server2/location2/;
            index index.svr2-loc2.htm;
        }

        location = /404.html {
            root myweb/;
            index 404.html;
        }
    }
}
```

```
#./sbin/nginx -c conf/nginx02.conf
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /opt/X_nginx/Nginx/conf/nginx02.conf:1
.
├── 404.html
├── server1
│   ├── location1
│   │   └── index.svr1-loc1.htm
│   ├── location2
│   │   └── index.svr1-loc2.htm
│   └── log
│       └── access.log
└── server2
    ├── location1
        │   └── index.svr2-loc1.htm
            ├── location2
                │   └── index.svr2-loc2.htm
                    └── log
                            └── access.log

                            8 directories, 7 files

```
## 测试myServer1的访问

```
http://myserver1:8081/server1/location1/
this is server1/location1/index.svr1-loc1.htm

http://myserver1:8081/server1/location2/
this is server1/location1/index.svr1-loc2.htm
```

## 测试myServer2的访问
```
http://192.168.0.254:8082/server2/location1/
this is server2/location1/index.svr2-loc1.htm

http://192.168.0.254:8082/svr2/loc2/
this is server2/location1/index.svr2-loc2.htm

http://192.168.0.254:8082/server2/location2/
404 404 404 404
```
# Nginx服务器架构

## 模块化结构

> Nginx 服务器的开发`完全`遵循模块化设计思想

### 什么是模块化开发？

1. 单一职责原则，一个模块只负责一个功能
2. 将程序分解，自顶向下，逐步求精
3. 高内聚，低耦合

### Nginx的模块化结构

+ 核心模块:         Nginx最基本最核心的服务，如进程管理、权限控制、日志记录；
+ 标准HTTP模块:     Nginx服务器的标准HTTP功能；
+ 可选HTTP模块:     处理特殊的HTTP请求
+ 邮件服务模块:     邮件服务
+ 第三方模块:       作为扩展，完成特殊功能


## Nginx的模块清单

+ 核心模块
    - ngx_core
    - ngx_errlog
    - ngx_conf
    - ngx_events
    - ngx_event_core
    - ngx_epll
    - ngx_regex

+ 标准HTTP模块
    - ngx_http
    - ngx_http_core             #配置端口，URI分析，服务器相应错误处理，别名控制(alias)等
    - ngx_http_log              #自定义access日志
    - ngx_http_upstream         #定义一组服务器，可以接受来自proxy, Fastcgi,Memcache的重定向；主要用作负载均衡
    - ngx_http_static
    - ngx_http_autoindex        #自动生成目录列表
    - ngx_http_index            #处理以`/`结尾的请求，如果没有找到index页，则看是否开启了`random_index`；如开启，则用之，否则用autoindex
    - ngx_http_auth_basic       #基于http的身份认证(auth_basic)
    - ngx_http_access           #基于IP地址的访问控制(deny,allow)
    - ngx_http_limit_conn       #限制来自客户端的连接的响应和处理速率
    - ngx_http_limit_req        #限制来自客户端的请求的响应和处理速率
    - ngx_http_geo
    - ngx_http_map              #创建任意的键值对变量
    - ngx_http_split_clients
    - ngx_http_referer          #过滤HTTP头中Referer为空的对象
    - ngx_http_rewrite          #通过正则表达式重定向请求
    - ngx_http_proxy
    - ngx_http_fastcgi          #支持fastcgi
    - ngx_http_uwsgi
    - ngx_http_scgi
    - ngx_http_memcached
    - ngx_http_empty_gif        #从内存创建一个1×1的透明gif图片，可以快速调用
    - ngx_http_browser          #解析http请求头部的User-Agent 值
    - ngx_http_charset          #指定网页编码
    - ngx_http_upstream_ip_hash
    - ngx_http_upstream_least_conn
    - ngx_http_upstream_keepalive
    - ngx_http_write_filter
    - ngx_http_header_filter
    - ngx_http_chunked_filter
    - ngx_http_range_header
    - ngx_http_gzip_filter
    - ngx_http_postpone_filter
    - ngx_http_ssi_filter
    - ngx_http_charset_filter
    - ngx_http_userid_filter
    - ngx_http_headers_filter   #设置http响应头
    - ngx_http_copy_filter
    - ngx_http_range_body_filter
    - ngx_http_not_modified_filter

+ 可选HTTP模块
    - ngx_http_addition         #在响应请求的页面开始或者结尾添加文本信息
    - ngx_http_degradation      #在低内存的情况下允许服务器返回444或者204错误
    - ngx_http_perl
    - ngx_http_flv              #支持将Flash多媒体信息按照流文件传输，可以根据客户端指定的开始位置返回Flash
    - ngx_http_geoip            #支持解析基于GeoIP数据库的客户端请求
    - ngx_google_perftools
    - ngx_http_gzip             #gzip压缩请求的响应
    - ngx_http_gzip_static      #搜索并使用预压缩的以.gz为后缀的文件代替一般文件响应客户端请求
    - ngx_http_image_filter     #支持改变png，jpeg，gif图片的尺寸和旋转方向
    - ngx_http_mp4              #支持.mp4,.m4v,.m4a等多媒体信息按照流文件传输，常与ngx_http_flv一起使用
    - ngx_http_random_index     #当收到/结尾的请求时，在指定目录下随机选择一个文件作为index
    - ngx_http_secure_link      #支持对请求链接的有效性检查
    - ngx_http_ssl              #支持https
    - ngx_http_stub_status
    - ngx_http_sub_module       #使用指定的字符串替换响应中的信息
    - ngx_http_dav              #支持HTTP和WebDAV协议中的PUT/DELETE/MKCOL/COPY/MOVE方法
    - ngx_http_xslt             #将XML响应信息使用XSLT进行转换

+ 邮件服务模块
    - ngx_mail_core
    - ngx_mail_pop3
    - ngx_mail_imap
    - ngx_mail_smtp
    - ngx_mail_auth_http
    - ngx_mail_proxy
    - ngx_mail_ssl

+ 第三方模块
    - echo-nginx-module         #支持在nginx配置文件中使用echo/sleep/time/exec等类Shell命令
    - memc-nginx-module
    - rds-json-nginx-module     #使nginx支持json数据的处理
    - lua-nginx-module

## Nginx的web请求处理机制

作为服务器软件，必须具备并行处理多个客户端的请求的能力， 工作方式主要以下3种：

+ 多进程(Apache)
    - 优点: 设计和实现简单；子进程独立
    - 缺点: 生成一个子进程要内存复制, 在资源和时间上造成额外开销
+ 多线程(IIS)
    - 优点: 开销小
    - 缺点: 开发者自己要对内存进行管理;线程之间会相互影响
+ 异步方式(Nginx)

经常说道异步非阻塞这个概念， 包含两层含义：

通信模式：
    + 同步: 发送方发送完请求后，等待并接受对方的回应后，再发送下个请求
    + 异步: 发送方发送完请求后，不必等待，直接发送下个请求

## Nginx的事件驱动模型

# 应用
## 架设简单文件服务器

将 /data/public/ 目录下的文件通过 nginx 提供给外部访问
```
#mkdir /data/public/
#chmod 777 /data/public/
```
```
worker_processes 1;
error_log logs/error.log info;
events {
    use epoll;
}
http {
     
    server {
            # 监听 8080 端口
            listen 8080;
            location /share/ {
                        # 打开自动列表功能，通常关闭
                        autoindex on;
                        # 将 /share/ 路径映射至 /data/public/，请保证 nginx 进程有权限访问 /data/public/
                        alias /data/public/;
                    }
       }
}
```