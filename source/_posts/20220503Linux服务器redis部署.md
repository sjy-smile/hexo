---
abbrlink: 24593a1d
title: Linux服务器redis部署
tags: redis
categories: 后端
date: 2023-05-03 16:22:35
cover:
---
## Redis安装和配置

1、首先在官网下载好`redis-6.0.9.tar.gz` [http://redis.io/](https://redis.io/)

2、下载使用上传到阿里云，这里时放到  `/usr/localhost/java/ ` 目录下

3、进入到 `/usr/localhost/java/ `目录，开始解压安装

```bash
tar -zxvf redis-6.0.9.tar.gz

#进入到redis-6.0.9目录执行make命令
make
#注意：这里执行make的时候可能会报错，这是可能没有gcc的原因，需要安装，命令为
yum install gcc-c++

#再次执行make命令，执行时先清除上次没安装成功的一些make内容
make distclean
make 
make install

#注意：这里gcc版本过低，也会导致安装不成功

#查看gcc版本
gcc -v  
#升级gcc版本，依次执行已下命令升级gcc
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
#以上为临时使用，需要长期使用需要执行一下命令
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

4、安装完成之后会默认在`/usr/local/bin`下生成一下 `redis` 的可执行文件，为了方便可以在 `redis-6.0.9` 建一个 `bin` 目录，把 `redis` 这些可执行文件都放到bin目录下，方便执行。

执行`./redis-server`命令

出现 `Ready to accept connections`，说明已经启动成功

5、连接redis执行`./redis-cli`

6、这时 `redis` 已经安装成功，我们来进行设置 `redis.conf` 配置文件：

- `redis` 默认是本机访问，其他地方无法连接，需要把这个注释掉`#bind 127.0.0.1`改成`bind 0.0.0.0` 

- 设置后台启动 `redis` 服务 `daemonize` 设置为 `yes`

- 执行后台启动`./redis-server redis.conf`

- `appendonly` 是 `redis` 持久化默认为 `no`，需要改为 `yes`

- `redis` 远程连接后，输入命令出现 `Error：Server closed the connection`，不需要登陆密码，改为 `no`

  ```bash
  protected-mode no
  ```

```bash
#常用命令
#后台启动的两种方式
./redis-server redis.conf 
nohup ./redis-server redis.conf &
nohup ./redis-server  redis.conf  >> /usr/local/java/redis-6.0.9/logs/redis.log  2>&1 &
#后台验证redis是否在启动
ps -ef |grep redis 或 ps aux | grep redis
#查看端口是否在监听
netstat -lntp | grep 6379
#关闭客户端
redis-cli shutdown
```

- Redis持久化报错

```bash
redis.exceptions.ResponseError: MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
```

```bash
#将stop-writes-on-bgsave-error设置为no
127.0.0.1:6379> config set stop-writes-on-bgsave-error no
或者进入redis.conf改no
```

- 注意：如果 `redis` 不设置登录密码在服务器上会出现 `key` 丢失情况，设置密码

- 在 `redis` 中所有的 `key` 都变成 `backup` 是因为 `redis` 暴漏在公网ip下，没有设置密码，遭到恶意请求

- 如果出现一下错误修改 `protected-mode  yes` 改为：`protected-mode no`，密码太短也有可能会出现这个问题

```java
org.springframework.data.redis.RedisConnectionFailureException: Unable to connect to Redis; nested exception is io.lettuce.core.RedisConnectionException: Unable to connect to 192.168.43.129:6379
```

## Redis Docker 安装测试

docker官方文档安装方式：`https://hub.docker.com/_/redis`

```bash
# 拉取最新镜像
docker pull redis:latest
# 启动
docker run --name myredis -p6379:6379 --restart=always -d redis:latest --requirepass "test123456"
# redis 使用命令行设置密码和持久化方式
docker run --name myredis -v /data/redis/data:/data \
-p 6379:6379 -d redis:latest --requirepass "test123456" --appendonly yes 

# 指定配置文件启动
docker run --name myredis -v /data/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data -p 6379:6379 -d redis:latest redis-server /etc/redis/redis.conf --requirepass "test123456" --appendonly yes 



# redis 使用自定义配置方式启动，如果要修改配置文件直接修改
docker run --name myredis \
-v /data/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data -p 6379:6379 -d redis:latest redis-server /etc/redis/redis.conf 
```






