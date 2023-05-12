---
abbrlink: d68e7f96
title: Linux服务器node环境配置
tags: node
categories: 前端
date: 2023-05-01 16:22:35
cover:
---
服务器用的是阿里云 `Centos7.9`

`node`官网：https://nodejs.org/en

镜像下载地址：https://nodejs.org/download/release

选择需要版本的 node，在服务器指定目录下进行一下命令下载（不建议安装最新版本）：

```bash
wget https://nodejs.org/download/release/latest-v17.x/node-v17.9.1-linux-x64.tar.gz
```

解压

```bash
tar -zxvf node-v17.9.1-linux-x64.tar.gz
```

配置环境变量

```bash
# 编辑 profile 文件
vim /etc/profile
# node 环境变量，追加以下内容
export PATH=$PATH:/data/node/node-v17.9.1-linux-x64/bin
# 保存并刷新配置
source /etc/profile
```

查看版本号

```bash
node -v
npm -v
```

出现版本号则说明配置成功

注意：查看版本号时出现以下错误，说明该操作系统的GLIBC版本低于Node所能支持的版本，建议16.x的版本，不建议安装最新版本

```bash
root@aliyun ~]# node -v
node: /lib64/libm.so.6: version `GLIBC_2.27' not found (required by node)
node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by node)
node: /lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found (required by node)
node: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by node)
node: /lib64/libc.so.6: version `GLIBC_2.28' not found (required by node)
node: /lib64/libc.so.6: version `GLIBC_2.25' not found (required by node)
```






