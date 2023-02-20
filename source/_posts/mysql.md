---
title: mysql
date: 2023-02-02 16:41:35
tags: test
categories: mysql
cover: /img/default_cover1.jpg
---

### MySql使用rpm的方式安装

官方下载地址：https://downloads.mysql.com/archives/community/

1、安装 MySQL 官方的 yum repository

```bash
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
 #　或者可以更改自家下载后的文件名，命名为,mysql.rpm
wget -O mysql.rpm http://repo.mysql.com//mysql57-community-release-el7-10.noarch.rpm
```

说明：`CentOS 7`的`yum`源中默认是没有`mysql`的。所以，为了解决这个问题我们首先下载安装`mysql`的`repository`源

2、下载 rpm 包

```bash
yum -y install mysql57-community-release-el7-10.noarch.rpm
```

==注意==：添加mysql-server rpm包（当前版本是 MySQL5.7）

`rpm -ivh https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm`

3、安装 MySQL 服务 

```bash
# 安装 MySql 服务方式一：
yum -y install mysql-community-server
# 安装 MySql 服务方式二(共3步)：
# 1、安装 mysql-server
yum install mysql-server
# 2、安装 mysql-devel
yum install mysql-devel
# 3、安装 mysql
yum install mysql
# 1、2、3步安装的过程选择的步骤都选y
# 查看 MySQL 安装的软件
rpm -qa|grep -i mysql
```

4、启动MySQL服务

```bash
# 启动服务 
systemctl start mysqld.service  或者  service mysqld start
# 停止服务
systemctl stop mysqld.service  或者 service mysqld stop
# 查看状态
systemctl status mysqld.service  或者  service mysqld status
#　重启服务
systemctl restart mysqld.service  或者 service mysqld restart
# 查看 mysql 启动的 pid
pidof mysqld
```

5、设置 MySql 开机启动

```bash
# 设置开机启动
systemctl enable mysqld
# 刷新配置
systemctl daemon-reload
```

6、关于登录 MySql，登录命令（==第一种方式使用密码登录==）

```bash
mysql -u root -p 
```

第一次启动MySQL后，就会有临时密码，这个默认的初始密码在/var/log/mysqld.log文件中，我们可以用以下命令来查看：

```bash
# 查看密码
[root@izuf61151k3ad2dso6mo9oz mysql]# grep "password" /var/log/mysqld.log
2021-04-21T14:41:47.850679Z 1 [Note] A temporary password is generated for root@localhost: Tt;vkIhrd71?
2021-04-21T14:44:16.590080Z 2 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2021-04-21T14:44:30.478638Z 3 [Note] Access denied for user 'root'@'localhost' (using password: YES)
# 进入 mysql 修改 root 用户的密码
update mysql.user set authentication_string=password('shijinying123!@#') where user='root';
# 修改之后刷新
flush privileges;
# 输入命令会报错误，或者先执行刷新语句，在执行修改密码语句
mysql> flush privileges;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
# 出现这种情况一次执行一下命令
SET PASSWORD = PASSWORD('Shijinying123!@#');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
flush privileges;
# 这里需要说明一下：密码必须包含数字，字母包含大小写，标点符号。不然是不能通过的。
# 完成之后就可以使用新密码登录了
mysql -u root -p Shijinying123!@#
# 修改 root 用户远程 连接
update user set host = '%' where user ='root';  #　直接修改 root 用户
# 修改完刷新数据库
flush privileges;
# 新增一条
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Shijinying123!@#' WITH GRANT OPTION;
```

==配置文件说明：==

> /etc/my.cnf 这是mysql的主配置文件
> /var/lib/mysql mysql数据库的数据库文件存放位置
> /var/log mysql数据库的日志输出存放位置

一下常用命令：

```bash
# 检查 MySQL 是否已经安装
yum list installed | grep mysql
# 已经安装的使用 yum 方式卸载
yum -y remove mysql-libs.x86_64   删除和MySQL有关的所有依赖  yum -y remove mysql*
# 验证下是否成功添加到了你的系统 repository 列表中
yum repolist enabled | grep "mysql.*-community.*" 
# 查看 MySQL 版本
yum repolist all | grep mysql
# 查看当前的启动的 MySQL 版本
yum repolist enabled | grep mysql
# 查看 MySQL 的安装目录
whereis mysql
# 查看 mysql 启动的 pid
pidof mysqld
# 查看 MySQL 3306 端口是否启动成功
netstat -nap |grep 3306
```

### MySql使用Docker的方式安装

1、拉去镜像

```bash
docker pull mysql:5.7
```

2、启动 mysql镜像

说明：-d 后台启动，-i 即使没有连接，也要保持标准输入保持打开状态，一般与 -t 连用，-t 分配一个伪tty，一般与 -i 连用.

```bash
docker run -itd --name mysql-test \
-v /data/mysql/mysql-data/:/var/lib/mysql \
-v /data/mysql/conf/my.cnf:/etc/mysql/my.cnf \
--privileged=true -e MYSQL_ROOT_PASSWORD=test123456 \
-p 3306:3306 --restart=always mysql:5.7

# 修改命令
docker update 
```

my.cnf 配置文件：

```bash
# Copyright (c) 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

[mysql]
default_character_set=utf8
[client]
default_character_set=utf8

[mysqld]
max_connections = 3000
max_user_connections = 3000
max_connect_errors = 100000
max_allowed_packet = 1G
character_set_server=utf8
collation-server=utf8_general_ci


sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

# bin-log
expire_logs_days = 15
binlog_format=mixed
server-id=168168

##########################
## log slow
###########################
slow_query_log=off
slow_query_log_file=/var/log/mysql/mysqld_slow.log
long_query_time=1
log_queries_not_using_indexes=on

##########################
## innodb
###########################
innodb_file_per_table=1
innodb_log_file_size=1024M
innodb_log_buffer_size=64M 

table_open_cache = 8000

```


