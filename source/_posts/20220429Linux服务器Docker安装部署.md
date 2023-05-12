---
abbrlink: 76bca4ef
title: Linux服务器Docker安装部署
tags: docker
categories: server
date: 2023-04-29 16:22:35
cover:
---
#### 1、虚拟化容器技术——Docker的安装及常用命令

官网地址：[链接](https://docs.docker.com/engine/install/centos/)

```bash
# 更新 yum 的索引
# yum 包更新到最新
yum update 或者 yum -y update
# 安装前先删除 docker 相关的包
# 使用 yum remove docker* 和一下命令是一样的
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 更新之后最简单的安装方式 (或者使用以方式)
yum install docker
# 安装需要的软件包，yum-util 提供 yum-config-manager 功能，另外两个是 devicemapper 驱动依赖的
yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置 yum 源为阿里云
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装 docker，不指定版本版本好默认为最新版本
yum install -y docker-ce docker-ce-cli containerd.io
#　安装后查看 docker 版本
docker -v
# 安装加速镜像
sudo mkdir -p /etc/docker
# 通过修改 daemon 配置文件 /etc/docker/daemon.json 来使用加速器
#　镜像地址：https://k7da99jp.mirror.aliiyuncs.com
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://k7da99jp.mirror.aliiyuncs.com"]
}
EOF
# 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

补充说明：

#### 2、docker 相关命令

```bash
# 查看 docker 版本
docker version
# 启动 docker
systemctl start docker
# 停止 docker
systemctl stop docker
# 重启 docker
systemctl restart docker
# 查看 docker 状态
systemctl status docker
# 设置开机启动
systemctl enable docker
systemctl unenable docker
# 查看 docker 相关信息
docker info
# 查看 docker 一些帮助
docker --help

# 运行 docker 镜像
docker run -d image-name
# 启动一个容器
docker start container_id or container_name 
# 停止运行容器
docker stop container_id or or container_name 
# 查看所有镜像
docker image ls 
docker container ls -a
# 查看所有正在运行的容器
docker ps -a
# 查看最近运行的容器
docker ps -l

# 查看镜像、容器、数据卷所占用的空间
docker system df

# docker stats 查看镜像内存、cpu的使用情况
docker stats

# 查看日志 
docker logs -f myredis
docker logs -f 容器id
```

##### 2-1、虚悬镜像

在镜像列表中，可以看到一个特殊的镜像，这个镜像既没有仓库名，也没有标签，均为 `<none>`

```bash
<none>               <none>              00285df0df87        5 days ago          342 MB
```

这个镜像原本是有镜像名和标签的，原来为 `mongo:3.2`，随着官方镜像维护，发布了新版本后，重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)** ，可以用下面的命令专门显示这类镜像：

```bash
docker image ls -f dangling=true

# 输出如下
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              00285df0df87        5 days ago     
```

一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除。

```bash
docker image prune
```

##### 2-2、用 docker image ls 命令来配合

像其它可以承接多个实体的命令一样，可以使用 `docker image ls -q` 来配合使用 `docker image rm`，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。

比如，我们需要删除所有仓库名为 `redis` 的镜像：

```bash
docker image rm $(docker image ls -q redis)
```

或者删除所有在 `mongo:3.2` 之前的镜像：

`docker image rm $(docker image ls -q -f before=mongo:3.2)`

#### 3、docker 操作容器

**启动容器**

所需要的命令主要为 `docker run`

```bash
# 拉去镜像
docker pull nginx
# 查看启动命令
docker run --help
# 启动镜像
docker run --name mynginx -p 80:80 -d nginx
# 修改 nginx 开机启动 注意：update不能修改端口
docker update --restart=always
# 进入容器
docker exec -it mynginx bash

# 如果进入容器不能编辑文件，执行一下命令
apt-get update 
apt-get install -y vim
```

**删除容器**

```bash
# 删除一个镜像
docker rmi image-id or image-name
# 删除所有镜像
docker rmi $(docker images -q)
# 强制删除所有镜像
docker rmi -r $(docker images -q)
# 删除所有镜像
docker rmi $(docker images -q -f dangling=true)
# 删除所有容器 (停止的镜像文件)
docker rm $(docker ps -a -q)
# 强制删除正在运行中的一个容器
docker rm -f 镜像名|id
# 查看所有数据卷
docker volume ls
# 删除所有数据卷
docker volume rm [volume_name]
# 删除所有未关联的数据卷
docker volume rm $(docker volume ls -qf dangling=true)
# 删除容器
docker container rm 容器id|镜像名字   或者 docker rm 容器id|镜像名字
# 清除所有处于终止状态的容器，用 docker container ls -a 查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。
# 注意：数据存储的地方
docker container prune
```

#### 4、Docker 提交、修改内容、传输

**1、修改容器内部内容**

```bash
# 启动 nginx
docker run --name=mynginx -p 80:80 -d nginx
# 进入容器
docker exec -it mynginx /bin/bash
# 进入index.html 文件夹
cd /usr/share/nginx/html
# 修改首页内容后，再次访问，这时首页内容已经改变
echo "<h1>Welcom come ningx</h1>" > index.html
```

**2、提交容器内容**

```bash
# 提交到docker容器
docker commit -a "ss"  -m "首页内容变化" mynginx mynginx:v1.0
# docker iamges 查看 mynginx:v1.0 镜像 
# 将镜像保存成压缩包 默认复制到当前所在文件夹
docker save -o  mynginx.tar mynginx:v1.0
```

**3、镜像传输**

```bash
# 将打包好的镜像传输到另一台机器
scp mynginx.tar root@101.133.170.222:/data/nginx
# 执行以上命令输入密码即可传输，注意：传输比较慢
# 加载镜像
docker load -i mynginx.tar
# docker images 查看已加载的镜像
# 启动 访问ip查看即可
docker run --name mynginx -p 80:80 -d mynginx:v1.0
```

**4、镜像挂在**

```bash
# 把容器内的文件复制到外边
docker cp eaeda5ac249d:/etc/nginx/nginx.conf /data/nginx/conf/
# 把外边目录下的内容复制到容器内
docker cp /data/nginx/conf/nginx.conf eaeda5ac249d:/etc/nginx/ 

# 容器启动文件挂在 :ro 表示容器内部只是可读(一般时把容器外的文件挂在到容器内，方便修改)
docker run  -p 80:80 \
-v /data/nginx/html:/usr/share/nginx/html:ro \
-v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
--name mynginx \
-d nginx
```

**5、redis 安装测试**

docker官方文档安装方式：`https://hub.docker.com/_/redis`

```bash
# redis 使用命令行设置密码和持久化方式
docker run --name myredis -v /date/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data -p 6379:6379 -d redis:latest --requirepass "123456" --appendonly yes 

# redis 使用自定义配置方式启动，如果要修改配置文件直接修改
docker run --name myredis \
-v /data/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data -p 6379:6379 -d redis:latest redis-server /etc/redis/redis.conf 
```

**6、定制镜像 Dockerfile**

把打好的 jar 包上传到服务器上

编辑 DockerFile 复制一下内容

```bash
# 该镜像需要依赖的基础镜像 或者 FROM java:8
FROM openjdk:8
# 将当前目录下的jar包复制到docker容器的/目录下
ADD java-demo-0.0.1-SNAPSHOT.jar /app.jar
# 声明服务运行在8088端口
EXPOSE 8080
# 指定docker容器启动时运行jar包
ENTRYPOINT ["java", "-jar","/app.jar"]
# 指定维护者的名字
MAINTAINER ss
```

构建镜像：`docker build -t my-demo:v1.0 -f DockerFile . `

启动构建完的镜像：`docker run --name mydemo -p 8001:8080 -d my-demo:v1.0`

访问：`http://ip:8001/hello`

**7、卸载 docker 和相关依赖**

```bash
# 1、卸载相关的依赖
yum remove docker-ce docker-ce-cli containerd.io
# 2、删除相关的资源
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

#### 5、docker 图形化工具 portainer

Github 地址：`https://github.com/portainer/portainer`

直接使用 docker 来安装 portainer 是非常方便的，仅需要两步即可完成。

5-1、首先下载 portainer 的 docker 镜像

```bash
docker pull portainer/portainer
```

5-1、再使用如下命令运行 portainer 容器

官网部署文档：`https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/`

```bash
docker run -p 9000:9000 -p 8000:8000 --name portainer \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /mydata/portainer/data:/data \
-d portainer/portainer:latest
```

第一次登录的时候需要创建管理员账号，访问地址：http://ip:9000/

#### 错误问题

出现以下错误 `-bash: /usr/bin/yum: No such file or directory` 更换yum源

到 `http://mirrors.kernel.org/centos/` 或者 `http://mirrors.163.com/centos/` 找对应系统版本号和系统位数下载，找到 `yum、yum-plugin-fastestmirror、yum-metadata-parser、python-urlgrabber` 这四个软件包下载

下载命令 直接用 wget 下载 rpm 包，然后执行下面三条命令

```bash
rpm -ivh  --nodeps yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm 
rpm -ivh  --nodeps yum-metadata-parser-1.1.4-10.el7.x86_64.rpm 
rpm -ivh  --nodeps yum-3.4.3-158.el7.centos.noarch.rpm
```

安装 nginx 出现以下问题，说明挂载的文件夹不存在，需要先把文件创建好

```bash
[root@aliyun data]# docker run --name mynginx -p 80:80 -p 443:443 -v /data/nginx/conf/:/etc/nginx/nginx.conf --privileged=true -v /data/nginx/log:/var/log/nginx -e TZ="Asia/Shanghai" --restart=always -d nginx:latest
8fe34ac4b3793b486ccf37ce9d86210a803e1de90df4727e440d9740983e8409
/usr/bin/docker-current: Error response from daemon: oci runtime error: container_linux.go:290: starting container process caused "container init exited prematurely".
```






