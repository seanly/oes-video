# Jenkins 对接 OpenLDAP

众所周知，在研发软件过程中，需要很多系统协助才能高效的完成软件开发。如果每一个系统都独立管理一套用户系统，对于开发者来说就是一件折腾的事情。所以需要统一认证或者单点登录，统一各个系统之间的账号体系和认证。

OpenLDAP可以作为统一认证或者作为单点登录的用户管理后端。由于OpenLDAP 比较基础，很多系统默认都会支持

本视频主要演示 Jenkins 如何对接 OpenLDAP 服务，内容包括：
1. OpenLDAP的基础概念和搭建过程
1. OpenLDAP如何通过客户端测试查询条件？
1. Jenkins如何开启LDAP的认证和测试配置？

开始之前可以先下载 https://github.com/seanly/oes-video 代码库地址，本期视频目录 `episodes/001`

# OpenLDAP

## 基础概念

非官方文档：https://wiki.shileizcc.com/confluence/display/openldap/OpenLDAP
官方文档：https://www.openldap.org/

## 环境准备

在搭建前需要设计目录架构，为方便视频演示，视屏中的目录架构如下图：
![目录架构图](https://i.imgur.com/P05gh8t.png)

为了方便演示，采用了容器方式运行演示环境，环境配置文件如下：

`file: docker-compose.yml`
```yaml=
version: '3'
services:
  openldap:
    image: seanly/openldap:v1.0
    restart: unless-stopped
    ports:
      - 12389:389
    volumes:
      - ./data/oes/data/openldap:/data
    environment:
      OPENLDAP_SUFFIX: dc=opsbox
      OPENLDAP_DOMAIN: oes 
      OPENLDAP_ADMIN_PASSWORD: admin123
      OPENLDAP_CONFIG_PASSWORD: admin123
```

针对docker-compose管理环境常用命令有以下几个：

* 环境启动命令 `docker-compose up -d`
* 进入运行的容器 `docker-compose exec openldap bash`
* 查看容器日志 `docker-compose logs -f openldap`
* 释放环境 `docker-compose down`
* 停止容器 `docker-compose stop openldap`
* 删除容器 `docker-compose rm openldap`

上面使用的镜像是基于网上CentOS系统安装OpenLDAP的文章制作的一个用于演示的简单镜像，如果自己搭建环境可以参考 [osixia/docker-openldap](https://github.com/osixia/docker-openldap) 开源项目

## 配置测试

OpenLDAP 的查询命令是`ldapsearch`, 命令使用参见 [ldapsearch](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)，所以可以通过 ldapsearch 命令测试查询条件是否正确

通过 `docker-compose exec openldap bash` 可以进入容器测试查询命令

```bash=
# 也可以将ldapadmin换成'*'
ldapsearch -x -b dc=oes,dc=opsbox "(& (uid=ldapadmin) (objectClass=inetOrgPerson))"
```

# Jenkins 配置

参见：https://www.bilibili.com/video/BV1fp4y1r7Dd?p=4

## 准备环境



## 配置测试