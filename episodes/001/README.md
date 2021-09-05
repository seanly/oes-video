# Jenkins 如何对接 OpenLDAP ？

众所周知，在研发软件过程中，需要很多系统协助才能高效的完成软件开发。如果每一个系统都独立管理一套用户系统，对于开发者来说就是一件折腾的事情。所以需要统一认证或者单点登录，统一各个系统之间的账号体系和认证。

OpenLDAP可以作为统一认证或者作为单点登录的用户管理后端。很多系统默认都会支持 OpenLDAP 服务。

本期视频相关资源存放在 https://github.com/seanly/oes-video/tree/master/episodes/001 


# 基础概念

1. 非官方文档：https://wiki.shileizcc.com/confluence/display/openldap/OpenLDAP
1. 官方文档：https://www.openldap.org/
1. LDAP 协议入门 https://zhuanlan.zhihu.com/p/147768058

# 环境准备

在搭建前需要设计目录架构，为方便视频演示，视屏中的目录架构如下图：

![目录架构图](https://i.imgur.com/P05gh8t.png)

为了方便演示，采用了容器方式运行演示环境，

操作步骤：

1. 通过`环境启动`命令启动环境，通过`查看日志`命令查看启动日志
1. 通过`进入容器`命令，登录进入 `openldap` 容器
1. 在成功登录容器后，通过`ldap-create-user.sh`命令创建`ldapadmin`管理用户和`test`测试用户，并记录界面输出的初始密码
1. 在容器中，通过`ldap-create-group.sh`命令创建`admin`组和`test1`组


上面使用的镜像是基于网上CentOS系统安装OpenLDAP的文章制作的一个用于演示的简单镜像，如果自己搭建环境可以参考 https://github.com/osixia/docker-openldap 开源项目

# 查询条件测试

OpenLDAP 的查询命令是`ldapsearch`, 命令使用参见 https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html ，所以可以通过 ldapsearch 命令测试查询条件是否正确

通过 `docker-compose exec openldap bash` 可以进入 OpenLDAP 的容器环境测试查询命令

```bash=
# 也可以将ldapadmin换成'*'
ldapsearch -x -b dc=oes,dc=opsbox "(& (uid=ldapadmin) (objectClass=inetOrgPerson))"
```

# Jenkins 配置

环境访问地址 http://localhost:8080

配置关键项：

1. root DN `dc=opsbox`
2. user search base `ou=users,dc=oes`
3. user search filter `uid={0}`
4. group search base `ou=groups,dc=oes`
5. group search filter `(& (cn={0}) (objectclass=groupOfNames))`
6. group membership filter `member={0}`


# Docker Compose常用命令

* 环境启动 `docker-compose up -d openldap`
* 进入容器 `docker-compose exec openldap bash`
* 查看日志 `docker-compose logs -f openldap`
* 释放环境 `docker-compose down`
* 停止容器 `docker-compose stop openldap`
* 删除容器 `docker-compose rm openldap`

# 视频地址

链接: https://pan.baidu.com/s/1ubuAIfTGJhAXzPP7hlcQjA  密码: qcah