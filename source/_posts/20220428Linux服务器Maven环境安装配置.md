---
abbrlink: f12e0651
title: Linux服务器Maven环境安装配置
tags: maven
categories: 后端
date: 2023-04-28 16:22:35
cover:
---
阿里云服务器配置 `maven` 环境变量

阿里云 `maven` 镜像[地址](https://mirrors.aliyun.com/apache/maven/maven-3/)，根据自己需求下载对应版本

```bash
# 创建文件夹
mkdir maven
# 下载
wget https://mirrors.aliyun.com/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
# 解压
tar -zxvf apache-maven-3.6.3-bin.tar.gz
# 进入
cd  apache-maven-3.6.3

# 获取路径:
pwd
  
# 配置环境变量：
vim  /etc/profile
=====================================================
# 配置文件添加下面两行内容，自己maven安装目录，例如：/maven/apache-maven-3.6.3
export MAVEN_HOME=/maven/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin
# 刷新环境变量
source /etc/profile
# 查看maven版本
mvn -version
```

修改仓库默认地址，进入 `maven` 下的 `conf` 目录，编辑 `settings.xml` 文件，打开 `localRepository` 注释的仓库地址，修改自己存放的依赖地址即可

```xml
<localRepository>/path/to/local/repo</localRepository>
```

修改 `maven` 配置文件下的 `settings.xml` 镜像地址为阿里云的，替换 `mirrors` 以下配置

```xml
	<!--阿里云镜像1-->
    <mirror>
      <id>aliyunId</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/repository/central</url>
    </mirror>
     <!--阿里云镜像2-->
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
     <!--阿里云镜像3-->
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>
     <!--阿里云镜像4-->
     <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>https://central.maven.org/maven2</url>
    </mirror>
```






