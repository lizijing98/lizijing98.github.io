---
title: Docker 创建 mysql 容器并挂载数据目录
date: 2022-02-13 17:05:36
update: 2022-02-13 17:05:36
tags: 
  - Docker
  - 容器化
categories: 学习
cover: https://pic2.zhimg.com/v2-b3e6202f5356edebd8c8205623eef0f1_1440w.jpg?source=172ae18b
---

# Docker 创建 mysql 容器并挂载数据目录

## 1. 环境介绍

操作系统：

> ProductName:	macOS
> ProductVersion:	11.6
> BuildVersion:	20G165

Docker 版本

> Docker version 20.10.12, build e91ed57

mysql 镜像版本

> REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
> mysql        5.7       0712d5dc1b14   2 weeks ago   448MB

## 2.准备工作

### 2.1 创建本地挂载目录

在本地创建挂载 mysql 容器的数据的目录：

```shell
mkdir mysqlData
```

### 2.2 创建 mysql 配置文件

```shell
touch my.cnf
```

配置文件内容如下：

```shell
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
lower_case_table_names=1 #实现mysql不区分大小（开发需求，建议开启）
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
default-time_zone = '+8:00'

# 更改字符集 如果想Mysql在后续的操作中文不出现乱码,则需要修改配置文件内容
symbolic-links=0
character-set-server=utf8mb4
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
```

### 2.3 创建一个测试用的 sql 脚本

```sql
DROP DATABASE IF EXISTS `testdb`;
CREATE DATABASE `testdb` DEFAULT CHARACTER SET utf8mb4;
USE `testdb`;
DROP TABLE IF EXISTS `tb_user`;
CREATE TABLE `tb_user` (
  id BIGINT(12) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '用户 ID',
  username VARCHAR(255) NOT NULL UNIQUE COMMENT '用户名',
  password VARCHAR(255) NOT NULL COMMENT '用户密码',
  real_name VARCHAR(255) NULL COMMENT '真实姓名',
  sex VARCHAR(10) NULL COMMENT '性别',
  phone_number VARCHAR(255) NULL COMMENT '手机号码',
  email VARCHAR(255) NULL COMMENT '邮箱',
  description VARCHAR(255) NULL COMMENT '备注',
  create_time TIMESTAMP NOT NULL DEFAULT NOW() COMMENT '创建时间',
  update_time TIMESTAMP NULL COMMENT '更新时间',
  is_deleted TINYINT(1) NOT NULL DEFAULT 0 COMMENT '删除标记,0 代表未删除,1 代表已删除',
  is_disable TINYINT(1) NOT NULL DEFAULT 0 COMMENT '禁用标记,0 代表未禁用,1 代表已禁用',
  PRIMARY KEY pk_user_id (id) USING BTREE COMMENT '用户 ID 主键',
  UNIQUE INDEX idx_username (username) USING BTREE COMMENT '用户名索引'
) COMMENT '用户表' ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
INSERT
  `tb_user`(username, password, real_name)
VALUES('test1', 'test1', '测试');
```



## 3.创建 mysql 容器并挂载本地目录

创建命令如下：

```shell
docker run -d -p 13306:3306 --name testmysql1 -v ${PWD}/dockerImages/mysqlData:/var/lib/mysql -v ${PWD}/dockerImages/my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD="123456" mysql:5.7
```

命令参数说明：

```shell
-d #后台运行容器，并返回容器 ID
-p 13306:3306 #指定容器端口映射，冒号前为宿主机端口，冒号后为容器端口
--name testmysql1 #指定容器名
-v ${PWD}/dockerImages/mysqlData:/var/lib/mysql #指定挂载，将宿主机中的目录挂载至容器的数据目录
-v ${PWD}/dockerImages/my.cnf:/etc/mysql/my.cnf #指定挂载，将创建的 mysql 配置文件挂载至容器中
-e MYSQL_ROOT_PASSWORD="123456" #指定 mysql 的 root 密码
mysql:5.7 #指定镜像
```

查看容器中挂载的配置文件：

```shell
docker exec -it testmysql1 cat /etc/mysql/my.cnf
```

结果可见自定的配置文件已经挂载成功：

<img src="https://s2.loli.net/2022/02/13/6hqXbxkQsifOEoP.png" alt="image-20220213160753195" style="zoom:50%;" />

查看宿主机中指定的挂载目录发现已有数据：

<img src="https://s2.loli.net/2022/02/13/LHn1lrQKkSZYayf.png" alt="image-20220213161240086" style="zoom:50%;" />

拷贝测试 SQL 脚本至容器中

```shell
docker cp ${PWD}/test.sql testmysql1:/.
```

进入容器 mysql 中

```shell
docker exec -it testmysql1 mysql -uroot -p123456
```

执行脚本并查看数据

```mysql
source /test.sql;
use testdb;
select * from tb_user;
```

这时可见挂载数据目录中出现创建的测试数据库对应的目录：

![image-20220213164122712](https://s2.loli.net/2022/02/13/GjpzmEesN6qIaZP.png)

且里面已有数据：

![image-20220213164208326](https://s2.loli.net/2022/02/13/OhRaAuWycFDtvdo.png)

在以同样的配置创建第二个测试容器 testmysql2，进入容器内部查看数据库：

<img src="https://s2.loli.net/2022/02/13/wEOlJq38KVrHbZa.png" alt="image-20220213165920503" style="zoom:50%;" />

因为挂载了数据目录，所以 testmysql2 容器可以读取到之前 testmysql1 容器保存的数据，并且可以进行对应的操作。

![image-20220213170206270](https://s2.loli.net/2022/02/13/7r2sOyLYc96whKG.png)
