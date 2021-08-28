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

操作步骤：

1. 通过`环境启动`命令启动环境，通过`查看日志`命令查看启动日志
1. 通过`进入容器`命令，登录进入容器
1. 在成功登录容器后，通过`ldap-create-user.sh`命令创建`ldapadmin`管理用户和`test`测试用户，并记录界面输出的初始密码
1. 在容器中，通过`ldap-create-group.sh`命令创建`admin`组和`test`组


针对docker-compose工具管理环境常用命令有以下几个：

* 环境启动 `docker-compose up -d`
* 进入容器 `docker-compose exec openldap bash`
* 查看日志 `docker-compose logs -f openldap`
* 释放环境 `docker-compose down`
* 停止容器 `docker-compose stop openldap`
* 删除容器 `docker-compose rm openldap`

上面使用的镜像是基于网上CentOS系统安装OpenLDAP的文章制作的一个用于演示的简单镜像，如果自己搭建环境可以参考 https://github.com/osixia/docker-openldap 开源项目

## 配置测试

OpenLDAP 的查询命令是`ldapsearch`, 命令使用参见 https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html ，所以可以通过 ldapsearch 命令测试查询条件是否正确

通过 `docker-compose exec openldap bash` 可以进入 OpenLDAP 的容器环境测试查询命令

```bash=
# 也可以将ldapadmin换成'*'
ldapsearch -x -b dc=oes,dc=opsbox "(& (uid=ldapadmin) (objectClass=inetOrgPerson))"
```

# Jenkins 配置


## 准备环境

针对 Jenkins 环境搭建社区已经提供了很多种方式

视频参见：https://www.bilibili.com/video/BV1fp4y1r7Dd?p=4

本视频为了简化搭建过程，将制作一个包含 ldap 插件的镜像，通过容器的方式运行

`file: docker-compose.yml`
```yaml=
version: '3'
services:
  jenkins-master:
    build:
      context: .
      dockerfile: Dockerfile
    image: seanly/jenkins:lts
    restart: unless-stopped
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - ./data/oes/data/jenkins:/var/jenkins_home
    environment:
      JAVA_OPTS: >-
        -server
        -Xmx1g -Xms1g
        -XX:+UnlockExperimentalVMOptions -XX:+UseContainerSupport
        -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
        -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
        -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
        -Djenkins.install.runSetupWizard=false
        -Dhudson.model.LoadStatistics.clock=2000
        -Dhudson.model.ParametersAction.keepUndefinedParameters=true
        -Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Shanghai
        -Duser.timezone=Asia/Shanghai
        -Dcom.sun.jndi.ldap.connect.pool.timeout=300000
        -Dhudson.security.csrf.DefaultCrumbIssuer.EXCLUDE_SESSION_ID=true

```

`file: Dockerfile`

```dockerfile=
FROM jenkins/jenkins:lts

ENV JENKINS_SLAVE_AGENT_PORT 50000

RUN jenkins-plugin-cli --verbose --plugins ldap
```

操作步骤：

1. 先通过`构建环境`命令制作演示镜像
1. 通过`环境启动`命令启动演示环境，通过`查看日志`命令查看 Jenkins 启动状态
1. 通过浏览器访问 Jenkins 服务，访问地址 http://localhost:8080


针对docker-compose工具管理环境常用命令有以下几个：

* 构建环境 `docker-compose build`
* 环境启动 `docker-compose up -d`
* 进入容器 `docker-compose exec jenkins-master bash`
* 查看日志 `docker-compose logs -f jenkins-master`
* 释放环境 `docker-compose down`
* 停止容器 `docker-compose stop jenkins-master`
* 删除容器 `docker-compose rm jenkins-master`

## 配置测试

配置样例如下图：

![](https://i.imgur.com/F5eQFFC.png)
