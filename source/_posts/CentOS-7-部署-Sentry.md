---
title: CentOS 7 部署 Sentry
date: 2019-05-14 14:31:28
categories: Sentry
tags:
    - CentOS
    - Docker
    - Sentry
    - LDAP
description: 在CentOS 7部署 Sentry，并且配置LDAP
---

由于工作需要，需要部署一套`Sentry`环境用来统计前端的报错，遇到了很多坑，将部署的正确姿势记录下来，留作备份。<!--more-->

## 更新yum源

```bash
# 1. 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 2. 添加
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 或
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 3.生成缓存
yum makecache
```

## Docker相关

### docker-ce

#### 卸载旧版Docker

因为我装了一个比较低版本的docker，所以要先卸载
```bash
$ yum list installed | grep docker 

## remove掉列出来的docker相关
$ yum remove [docker]
```

#### 安装最新版Docker

```bash
$ curl -fsSL https://get.docker.com/ | sh

# centos 7
$ systemctl restart docker # 启动服务
$ systemctl enable docker # 开机启动

# centos 6
$ service docker restart # 启动服务
$ chkconfig docker on # 开机启动

# 查看安Docker版本
$ docker version # docker -v
```

#### 安装Docker-compose

> https://github.com/docker/compose/releases

去这个链接找到最新的安装脚本，比如

```bash
curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## Sentry 相关

### `clone`代码

> https://github.com/getsentry/onpremise

### 部署`Sentry`

按照`Up and Running`步骤来

```bash
# 1. 创建数据卷
docker volume create --name=sentry-data && docker volume create --name=sentry-postgres

# 2. 生成.env
cp -n .env.example .env

# 3. 构建
docker-compose build

# 4. 生成secret-key 并且将生成的key加入到.env文件
docker-compose run --rm web config generate-secret-key

# 5. 运行，这最后可以创建一个超级管理员用户
docker-compose run --rm web upgrade

# 6. 启动
docker-compose up -d

# 7. 访问
localhost:9000
```

### 其他命令

创建用户
```bash
docker-compose run --rm web createuser
```

查看日志
```bash
docker-compose logs web

## 实时日志
docker-compose logs --tail="1000" -f web
```

进入容器
```bash
docker-compose exec web bash
```

## `LDAP` 相关

> https://github.com/Banno/getsentry-ldap-auth

### 增加`LDAP`配置

```python
###########
# LDAP/AD #
###########

import ldap
from django_auth_ldap.config import LDAPSearch, GroupOfUniqueNamesType

AUTH_LDAP_SERVER_URI = 'ldap://example'
AUTH_LDAP_BIND_DN = 'cn=readuser,ou=ex,dc=example,dc=com'
AUTH_LDAP_BIND_PASSWORD = 'example'
OU=unicode('dc=example,dc=com', 'utf8')
AUTH_LDAP_USER_SEARCH = LDAPSearch(
    OU,
    ldap.SCOPE_SUBTREE,
    '(cn=%(user)s)',
)

AUTH_LDAP_GROUP_SEARCH = LDAPSearch(
    '',
    ldap.SCOPE_SUBTREE,
    '(objectClass=groupOfUniqueNames)'
)

AUTH_LDAP_GROUP_TYPE = GroupOfUniqueNamesType()
AUTH_LDAP_REQUIRE_GROUP = None
AUTH_LDAP_DENY_GROUP = None

AUTH_LDAP_USER_ATTR_MAP = {
    'name': 'cn',
    'email': 'email'
}

AUTH_LDAP_FIND_GROUP_PERMS = False
AUTH_LDAP_CACHE_GROUPS = True
AUTH_LDAP_GROUP_CACHE_TIMEOUT = 3600

AUTH_LDAP_DEFAULT_SENTRY_ORGANIZATION = u'sentry'
AUTH_LDAP_SENTRY_ORGANIZATION_ROLE_TYPE = 'member'
AUTH_LDAP_SENTRY_ORGANIZATION_GLOBAL_ACCESS = True
AUTH_LDAP_SENTRY_USERNAME_FIELD = 'uid'

AUTHENTICATION_BACKENDS = AUTHENTICATION_BACKENDS + (
    'sentry_ldap_auth.backend.SentryLdapBackend',
)
```

### 安装依赖

#### 更新容器`apt`源

进入容器
```bash
docker-compose exec web bash
```

因为容器内很干净，所以安装`vim`来编辑
```bash
apt install vim
```

查看`Linux`版本，可以看出是 `Debian 9`
```bash
cat /etc/issue

Debian GNU/Linux 9 \n \l
```

更新`apt`源，在`/etc/apt/sources.list`新增
```
deb http://mirrors.aliyun.com/debian stretch main contrib non-free
deb-src http://mirrors.aliyun.com/debian stretch main contrib non-free
deb http://mirrors.aliyun.com/debian stretch-updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian stretch-updates main contrib non-free
deb http://mirrors.aliyun.com/debian-security stretch/updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian-security stretch/updates main contrib non-free
```

update
```bash
apt-get update
```

#### 修改`Dockerfile`

修改`Dockerfile`文件，在下面加入
```docker
RUN apt-get update && apt-get install -y libsasl2-dev python-dev libldap2-dev libssl-dev
RUN pip install sentry-ldap-auth
```

#### 坑
`sentry-ldap-auth`作者推荐将插件加入到`requirements.txt`里面，但是这样会有问题，不要尝试

### 重启`Sentry`服务

分别执行上面`3`、`5`、`6`、`7`步骤登录的时候就可以用`ldap`账户登录了，但是登陆进去没有组织


## 参考
1. [Centos安装最新版Docker CE、Docker-Compose](https://www.zengjianfeng.com/2018/07/321.html)
2. [Debian9换源(阿里源)（Linux子系统）](https://www.cnblogs.com/dayfly5/p/10275353.html)