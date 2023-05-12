---
abbrlink: 7e10b77c
title: Linux服务器nginx部署及用配常置详解
tags: nginx
categories: 后端
date: 2023-05-04 16:22:35
cover:
---
## 安装 Nginx

1、tar 压缩包安装，安装 `nginx` 需要先将官网下载的源码进行编译，编译依赖 `gcc` 环境，如果没有 `gcc` 环境，则需要安装：

```
yum install gcc-c++
```

2、 `linux` 上安装 `pcre` 库，`PCRE(Perl Compatible Regular Expressions)` 是一个 `Perl` 库，包括 `perl` 兼容的正则表达式库。`nginx` 的 `http` 模块使用 `pcre` 来解析正则表达式，所以需要在 `linux` 上安装 `pcre` 库，`pcre-devel` 是使用 `pcre` 开发的一个二次开发库。`nginx` 也需要此库。命令：

```
yum install -y pcre pcre-devel
```

3、`zlib` 安装，`zlib` 库提供了很多种压缩和解压缩的方式， `nginx` 使用 `zlib` 对 `http` 包的内容进行 `gzip` ，所以需要在 `Centos` 上安装 `zlib` 库

```
yum install -y zlib zlib-devel
```

4、 在 `linux` 安装 `openssl` 库，`OpenSSL` 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 `SSL` 协议，并提供丰富的应用程序供测试或其它目的使用。`nginx` 不仅支持 `http` 协议，还支持 `https`（即在 `ssl` 协议上传输 `http`），所以需要在 `Centos` 安装 `OpenSSL` 库。

```
yum install -y openssl openssl-devel
```

1~4 使用一条命令执行就是：

```
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```

4、进入压缩包所在目录，使用以下命令进行解压

```
tar -zxvf nginx-1.6.2.tar.gz
```

5、使用命令：`cd nginx-1.6.2.tar.gz`，进入解压缩后文件夹，配置编译参数命令，并添加ssl模块(默认没有)，`Linux` 上使用命令。

6、使用命令：`cd nginx-1.6.2`，进入解压缩后文件夹，配置编译参数命令，并添加ssl模块(默认没有)，`Linux` 上使用命令和效果图如下：

```
# 可以直接运行 ./configure  命令
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_stub_status_module
```

7、在上步的基础上编译并安装，`Linux` 上使用命令和效果图如下(部分截图)，如果不按以上步骤会有两个错误，重新执行一遍即可

```
make && make install
# 安装到指定目录
make && make PREFIX=/usr/local/nginx install
```

**将 nginx 加到开机自启**(根据自己需要添加即可)

```
echo "/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf &" >>/etc/rc.local
```

## 卸载nginx

1、首先通过命令：`ps -ef|grep nginx`，查看 `nginx` 运行状态

2、如果 `nginx` 处于运行中的状态，使用命令：`kill -9` 进程号，杀掉进程

3、全局查找 `nginx` 相关的文件所处路径，使用以下命令进行查询

```
sudo find / -name nginx*
```

4、根据上一步查询出的 `nginx` 相关路径，就可以使用以下命令逐一删除

```
# nginx* 代表自该目录起所有的子目录
# 该条命令会删除/usr/local/路径下nginxd的整个文件夹(该文件的子文件都会删除掉)
rm -rf file /usr/local/nginx*

# 清除依赖
yum remove nginx
```

5、通过以上几步，基本可以完全卸载 `nginx` ，以便重新安装 `nginx`

## nginx常用命令

```
# 首先进入nginx目录
cd /usr/local/nginx
./nginx  # 启动
./nginx -s stop  # 停止(暴力停止服务)
./nginx -s quit  # 安全退出(优雅停止服务)
./nginx -s reload  # 重新加载配置文件
./nginx -h # 帮助命令
./nginx -t # 检查配置文件
/usr/local/nginx/conf/nginx.conf # nginx配置文件路径
ps aux|grep nginx  # 查看nginx进程
```

## Docker安装部署 nginx

`nginx` 文件挂载到虚拟机外部

1、运行 `nginx` ，把相关配置文件复制宿主机

```
docker run --name mynginx -p 80:80 -p 443:443 -d nginx:latest
```

2、创建挂在的文件夹

```
mkdir -p /data/nginx/conf /data/nginx/log /data/nginx/html /data/nginx/ssl
```

3、将容器 `nginx.conf` 文件复制到宿主机

```
docker cp mynginx:/etc/nginx/nginx.conf /data/nginx/conf/
```

4、将容器 `conf.d` 文件夹下内容复制到宿主机

```
docker cp mynginx:/etc/nginx/conf.d /data/nginx/conf/conf.d
```

5、将容器中的 `html` 文件夹复制到宿主机

```
docker cp mynginx:/usr/share/nginx/html /data/nginx/
```

6、强制删除正在运行的 `nginx`：`docker rm -f mynginx`

7、重新启动运行 `nginx`

```
docker run --name mynginx -p 80:80 -p 443:443 \
-v /data/nginx/html/:/usr/share/nginx/html \
-v /data/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /data/nginx/log:/var/log/nginx \
--privileged=true -e TZ="Asia/Shanghai" -v /data/nginx/ssl/:/etc/nginx/ssl/:rw \
--restart=always -d nginx:latest

docker run --name my_nginx -p 80:80 \
-v /usr/local/docker/nginx/conf/conf.d/default.conf:/etc/nginx/conf.d/default.conf \
-v /usr/local/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /usr/local/docker/nginx/html/:/usr/share/nginx/html/ \
-v /usr/local/docker/nginx/logs/:/var/log/nginx/ \
--privileged=true -d --restart=always nginx:latest


# 停止服务
docker stop mynginx
# 启动服务
docker start mynginx
# 重启服务
docker restart mynginx
# 强制删除并停止
docker rm -f mynginx
```

## nginx配置说明

`nginx` 的负载均衡策略有4种：

**轮询(默认)**

最基本的配置方法，它是 `upstream` 的默认策略，每个请求会按时间顺序逐一分配到不同的后端服务器。

参数有：

| 参数         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| fail_timeout | 与max_fails结合使用                                          |
| max_fails    | 设置在fail_timeout参数设置的时间内最大失败次数，如果在这个时间内，所有针对该服务器的请求都失败了，那么认为该服务器会被认为是停机了。 |
| fail_time    | 服务器会被认为停机的时间长度,默认为10s。                     |
| backup       | 标记该服务器为备用服务器。当主服务器停止时，请求会被发送到它这里。 |
| down         | 标记服务器永久停机了。                                       |

```
##定义Nginx运行的用户和用户组。window下不指定
#user  nobody;

#nginx进程数,建议设置为等于CPU总核心数.
worker_processes  1;

#全局错误日志定义类型,[ debug | info | notice | warn | error | crit ]
#error_log  /usr/local/nginx/log/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程文件
#pid        logs/nginx.pid;

#一个nginx进程打开的最多文件描述符数目,理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除,但是nginx分配请求并不均匀,所以建议与ulimit -n的值保持一致.
worker_rlimit_nofile 65535;

#工作模式与连接数上限
events {
     #参考事件模型,use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型,如果跑在FreeBSD上面,就用kqueue模型.
    use epoll;
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections 65535;
}

#设定http服务器 
http {
    include       mime.types; #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型

    #charset utf-8; #默认编码
    server_names_hash_bucket_size 128; #服务器名字的hash表大小
    client_header_buffer_size 32k; #上传文件大小限制
    large_client_header_buffers 4 64k; #设定请求缓
    client_max_body_size 8m; #设定请求缓
     
    # 开启目录列表访问,合适下载服务器,默认关闭.
    autoindex on; # 显示目录
    autoindex_exact_size on; # 显示文件大小 默认为on,显示出文件的确切大小,单位是bytes 改为off后,显示出文件的大概大小,单位是kB或者MB或者GB
    autoindex_localtime on; # 显示文件时间 默认为off,显示的文件时间为GMT时间 改为on后,显示的文件时间为文件的服务器时间
     
    sendfile on; # 开启高效文件传输模式,sendfile指令指定nginx是否调用sendfile函数来输出文件,对于普通应用设为 on,如果用来进行下载等应用磁盘IO重负载应用,可设置为off,以平衡磁盘与网络I/O处理速度,降低系统的负载.注意：如果图片显示不正常把这个改成off.
    tcp_nopush on; # 防止网络阻塞
    tcp_nodelay on; # 防止网络阻塞
     
    keepalive_timeout 120; # (单位s)设置客户端连接保持活动的超时时间,在超过这个时间后服务器会关闭该链接
     
    # FastCGI相关参数是为了改善网站的性能：减少资源占用,提高访问速度.下面参数看字面意思都能理解.
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
     
    # gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #允许压缩的页面的最小字节数,页面字节数从header偷得content-length中获取.默认是0,不管页面多大都进行压缩.建议设置成大于1k的字节数,小于1k可能会越压越大
    gzip_buffers 4 16k; #表示申请4个单位为16k的内存作为压缩结果流缓存,默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果
    gzip_http_version 1.1; #压缩版本（默认1.1,目前大部分浏览器已经支持gzip解压.前端如果是squid2.5请使用1.0）
    gzip_comp_level 2; #压缩等级.1压缩比最小,处理速度快.9压缩比最大,比较消耗cpu资源,处理速度最慢,但是因为压缩比最大,所以包最小,传输速度快
    gzip_types text/plain application/x-javascript text/css application/xml;
    #压缩类型,默认就已经包含text/html,所以下面就不用再写了,写上去也不会有问题,但是会有一个warn.
    gzip_vary on;#选项可以让前端的缓存服务器缓存经过gzip压缩的页面.例如:用squid缓存经过nginx压缩的数据
     
    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;

   #####################################################################################################
 ##upstream的负载均衡,四种调度算法##
    #调度算法1:轮询.每个请求按时间顺序逐一分配到不同的后端服务器,如果后端某台服务器宕机,故障系统被自动剔除,使用户访问不受影响
    upstream webhost {
        server localhost:8001 max_fails=3 fail_timeout=20s;
        server localhost:8002;
        server localhost:8003 backup;
        server localhost:8003 down;
        server localhost:8003 weight=2;
    }
    #调度算法2:weight(权重).可以根据机器配置定义权重.权重越高被分配到的几率越大
    upstream webhost {
        server localhost:8001 weight=2;
        server localhost:8002 weight=3;
    }
    #调度算法3:ip_hash. 每个请求按访问IP的hash结果分配,这样来自同一个IP的访客固定访问一个后端服务器,有效解决了动态网页存在的session共享问题
    upstream webhost {
        ip_hash;
        server localhost:8001;
        server localhost:8002;
    }
    #调度算法4:url_hash(需安装第三方插件).此方法按访问url的hash结果来分配请求,使每个url定向到同一个后端服务器,可以进一步提高后端缓存服务器的效率.Nginx本身是不支持url_hash的,如果需要使用这种调度算法,必须安装Nginx 的hash软件包
    upstream webhost {
        server localhost:8001;
        server localhost:8002;
        hash $request_uri;
    }

   # 如果要监听两个端口需要添加server内容
   server {
    
        listen       8080;
        # 域名可以有多个,用空格隔开
        server_name  127.0.0.1;
        server_name baidu.com;
        # HTTP 自动跳转 HTTPS
        # rewrite ^(.*) https://$server_name$1 permanent;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /usr/local/java/ry-v/qd;
   try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }
        
        #代理头
  location /prod-api/ {
   proxy_set_header Host $http_host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header REMOTE-HOST $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   # 后端代理地址
   proxy_pass http://101.133.170.222:8080/;
   # 多个路径部署 proxy_pass http://wenhost/;
  }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

 #虚拟主机的配置
    server {
    
        listen       80;
        # 域名可以有多个,用空格隔开
        server_name baidu.com;
        # HTTP 自动跳转 HTTPS
        rewrite ^(.*) https://$server_name$1 permanent;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /prod-api/ {
            root   html;
            index  index.html index.htm;
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


    # 
    HTTPS server
    #
    # 监听端口 HTTPS
    server {
        listen       443 ssl;
        server_name  www.nginx.cn;
        
        #ssl_certificate ./ssl/[server_name].pem; # 指定证书的位置，Linux上可以设置相对路径，Windows上要设置绝对路径
        #ssl_certificate_key ./ssl/[server_name].key; # 同上 
        #ssl_trusted_certificate ./ssl/[server_name].cer;
          
        # 配置域名证书
        ssl_certificate         ./ssl/fullchain.cer; 
        ssl_certificate_key     ./ssl/[server_name].key;
        ssl_session_cache    shared:SSL:1m;

        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 SSLv2 SSLv3;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        
        proxy_set_header    Host                  $host;
        proxy_set_header    X-Real-IP             $remote_addr;
        proxy_set_header    X-Forwarded-For       $proxy_add_x_forwarded_for;
     proxy_set_header    X-Forwarded-Proto     $scheme;

  location / {
          proxy_connect_timeout 1;
          proxy_pass http://www.nginx.cn;
        }
        
        # 配置地址拦截转发，解决跨域验证问题
        location /oauth/{
            proxy_pass https://localhost:8080/oauth/;
            proxy_set_header HOST $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
        # JS和CSS缓存时间设置
        location ~ .*\.(js|css)?$ {
            expires 1h;
        }
        # 日志格式设定
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';
        # 定义本虚拟主机的访问日志
        access_log /usr/local/nginx/log/access.log access;
        
        # 设定查看Nginx状态的地址.StubStatus模块能够获取Nginx自上次启动以来的工作状态，此模块非核心模块，需要在Nginx编译安装时手工指定才能使用
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file conf/htpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生.
        }
        error_page 404 500 502 503 504 /50x.html;
      }
}
nohup` 后台守护进程方式运行项目：`nohup java -jar project.jar >nohup.out 2>&1 &
```

## nginx常用命令

```
# 首先进入nginx目录
cd /usr/local/nginx
./nginx  # 启动
./nginx -s stop  # 停止(暴力停止服务)
./nginx -s quit  # 安全退出(优雅停止服务)
./nginx -s reload  # 重新加载配置文件
./nginx -h # 帮助命令
./nginx -t # 检查配置文件
/usr/local/nginx/conf/nginx.conf # nginx配置文件路径
ps aux|grep nginx  # 查看nginx进程
```




