---
title: mysql 搭建与基础
date: 2021-03-2 18:02:30
categories:
  - backend
  - mysql
tags:
  - backend
  - mysql
---

mysql基础学习与基础docker搭建

<!-- more -->

## 数据库相关定义

### 数据库

数据库 database，按照特定格式存储数据的**文件集合**

### 特点

用户可以对存储的数据进行增删改查操作。

### 数据库管理系统

用户与操作系统之间使用的维护数据库的数据管理软件，例如 mysql，mongodb，oracle 等。

### 数据库分类

分为关系型数据库和非关系型数据库，mysql 数据关系型

#### 关系型数据库是由多张表连接组成的数据库

优点：

- 表结构，格式一致，易于维护
- 提供成熟的 sql 语言操作，使用方便
- 支持事务，表关联外键，能充分保证数据安全与完整性
- 数据存储在硬盘中，丢失风险低

缺点：

- 数据存储在硬盘，读写性能较低，不能满足海量数据的高效率读写，经过优化可以提高一定查询速度，拆分表，表按照月度拆分等
- 只支持基础类型和少量的集合类型

对于高并发的场景，可以分库，建立 mysql 集群。

分布式和集群的区别：

- 分布式是指将不同业务分布到不同地方
- 集群是将几台服务器集中在一起，目的是为了同一个业务

#### 非关系性数据库 NoSql

优点：

- 支持存储格式较多，可以是 key-value，数组，文档形式，图片形式
- 速度快，更适合海量数据访问
- 支持分布式处理，一个数据库可以分成多个部分保存到不同服务器

缺点

- 非关系型数据库没有 sql 支持，使用不便，维护成本高
- 没有事务处理，没有表关联，所以无法保证数据完整性和安全性，不适合对安全要求较高的场景
- 功能相对关系型数据库会不够完善（随着发展，肯定会继续完善）

## mysql 数据库安装

### mysqld 和 mysql

- mysqld 在安装好 mysql 数据库后，身份是一个后台服务程序，mysql 启动后，mysqld 会开启一个守护进程，如果守护进程没有开启，mysql 服务器可能会挂掉
- mysql 相当于是客户端和 mysql 服务器之间进行 sql 语句交互提供操作环境的 cli 命令行工具，客户端连接 mysql 服务器和操作表都在这个操作环境下进行，参考一下前端开发中的脚手架，也是提供了操作的命令行工具

### docker 安装 mysql

#### 拉取 mysql 的镜像

mac m1 arm64v8 平台

```sh
docker pull arm64v8/mysql:latest
```

其他平台

```
docker pull mysql:latest
```

#### 运行容器

```sh
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=some_root_password -p 3306:3306  -d arm64v8/mysql:latest
```

#### 使用 navicat 测试 mysql 连接

填入对应账号密码，主机等，然后点击左下角的 test connection

![](/pics/infrastructure/test-mysql.jpeg)

测试成功后，点击右下角 save 保存连接

![](/pics/infrastructure/test-mysql-success.jpeg)

### my.ini/my.cnf 配置文件解析

现阶段一般都是使用 docker 搭建数据库等环境

本地环境

- 物理机：mac m1
- 虚拟机：docker
- 镜像：arm64v8/mysql

不同物理机平台可能会有些不同

在 docker 中，my.cnf/my.ini 位于/etc/my.cnf

```bash

[mysql]
# mysql客户端默认字符集
default-character-set=UTF8MB4

[mysqld]

skip-host-cache
skip-name-resolve
# mysql数据库的数据存储目录
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
pid-file=/var/run/mysqld/mysqld.pid
user=mysql
# 服务端使用的字符集,与客户端一样
character-set-server=UTF8MB4
# 最大连接数
max_connections=200
# 创建新表使用的默认存储引擎
default-storage-engine=INNODB

[client]
port=3306
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
```

### mysql 登录

进入镜像内部

```sh
docker exec -it <container-name> bash
```

输入 mysql -u root -p 后，提示输入密码，输入密码后就可以登录到 mysql 内部了

```sh
mysql -u root -p
```

## mysql 基本操作

操作需要成功登录 mysql 之后

### 显示所有数据库

```sh
show databases;
```

### 新建用户

创建一个没有权限用户

```sh
create user 'admin' identified with mysql_native_password by 'some_psw';
```

### 分配权限

分配所有权限给 admin

```sh
grant all privileges ON *.* TO admin@'%';
```

### 创建数据库

创建名称为 test 的数据库，如果数据库已存在，则不创建。

character: 指定数据库的字符集，避免存储的数据出现乱码，或者某些字符不支持

collate: 指定字符集的默认校对规则

```sh
create database IF NOT EXISTS test CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

### 查看数据库

显示所有数据库

```sh
show databases
```

### 删除数据库

如果 test 数据库存在，删除 test 数据库

```sh
drop database if exists test
```

### 切换数据库

会切换到 test 数据库

```sh
use test;
```

### 创建数据表

前面说过，关系型数据库是一张一张的表架构组合而成，userid，username，psw 等这些称为字段（field），而字段的类型从创建表阶段就定好了

userid int NOT NULL AUTO_INCREMENT
userid int 类型 非空 自增长

primary key(userid)，表示是一个可以唯一表示一条记录的字段

**注意：创建数据表之前必须使用 use \<database name\>去切换到对应的数据库**

**注意：创建数据表之前必须使用 use \<database name\>去切换到对应的数据库**

**注意：创建数据表之前必须使用 use \<database name\>去切换到对应的数据库**

```sh
create table userinfo(
  userid int NOT NULL AUTO_INCREMENT,
  username varchar(30) NOT NULL,
  psw int NOT NULL,
  address varchar(50) default 'empty address',
  valid TINYINT default 1,
  birth DATETIME null,
  PRIMARY KEY(userid)
);
```

创建数据表完成，输入 show tables;显示当前数据库的表

```sh
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| userinfo       |
+----------------+
1 row in set (0.00 sec)
```

### 修改表名

```sh
mysql> ALTER TABLE userinfo RENAME TO myuserinfo;
Query OK, 0 rows affected (0.04 sec)

mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| myuserinfo     |
+----------------+
1 row in set (0.00 sec)
```

### 添加数据到某个表 insert into

提供完整数据的写法

```sh
mysql> insert into userinfo values(1, 'username1', '123123', 'guangzhou', 1, '1989/1/1 01:01:01');
Query OK, 1 row affected, 1 warning (0.00 sec)
```

有些字段在创建数据表的阶段提供了默认值，例如地址 address，是否合法 valid，那么可以通过指定字段和对应值插入数据

```sh
mysql> insert into userinfo(username, psw, birth) values('user2', '123123123', '1989/1/1 01:01:01');
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> select * from userinfo;
+--------+-----------+-----------+---------------+-------+---------------------+
| userid | username  | psw       | address       | valid | birth               |
+--------+-----------+-----------+---------------+-------+---------------------+
|      1 | username1 |    123123 | guangzhou     |     1 | 1989-01-01 01:01:01 |
|      2 | user2     | 123123123 | empty address |     1 | 1989-01-01 01:01:01 |
+--------+-----------+-----------+---------------+-------+---------------------+
2 rows in set (0.00 sec)
```

某些字段指定了 not null 并且没有指定 default，则必须要提供值.

这个例子没有提供 username 字段的值，提示 username 没有默认值

```sh
mysql> insert into userinfo(psw, birth) values('123123', '1989/1/1 01:01:01');
ERROR 1364 (HY000): Field 'username' doesn't have a default value
```

### 选择查询 select

在 userinfo 表中查询所有数据

```sh
mysql> select * from userinfo;
+--------+-----------+--------+-----------+-------+---------------------+
| userid | username  | psw    | address   | valid | birth               |
+--------+-----------+--------+-----------+-------+---------------------+
|      1 | username1 | 123123 | guangzhou |     1 | 1989-01-01 01:01:01 |
+--------+-----------+--------+-----------+-------+---------------------+
1 row in set (0.00 sec)
```

### 修改字段名称

```sh
mysql> alter table myuserinfo change psw password varchar(20);
Query OK, 2 rows affected (0.05 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

### 更新一行数据

```sh
mysql> update userinfo set age=56 where userid=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

## mysql 数据类型

### 字符类型

- char：固定长度 1-255 字节，定好了长度之后，无论实际长度，都是占用固定长度
- varchar：可变长度 1-255 字节，定好了长度之后，会根据实际长度进行伸缩
- text：大文本 65535 字节

### 整数类型

- tinyint：1 byte
- smallint：2 byte
- mediumint：3 byte
- int：4 byte
- bigint：8 byte

### 浮动类型

- float：4 byte
- double：8 byte

### 日期/时间类型

- date：3 byte 记录的是年月日
- datetime：8 byte 年月日时分秒

## 数据库查询

### 查询所有行

select * from \<table name\>

```sh
mysql> select * from userinfo;
+--------+-----------+-----------+---------------+-------+---------------------+
| userid | username  | password  | address       | valid | birth               |
+--------+-----------+-----------+---------------+-------+---------------------+
|      1 | username1 | 123123    | guangzhou     |     1 | 1989-01-01 01:01:01 |
|      2 | user2     | 123123123 | empty address |     1 | 1989-01-01 01:01:01 |
+--------+-----------+-----------+---------------+-------+---------------------+
2 rows in set (0.02 sec)
```

### 投影查询，即查询局部字段

需要查询多个字段可以使用逗号分隔

select address from userinfo;
select userid,username from userinfo;

```sh
mysql> select userid,username from userinfo;
+--------+-----------+
| userid | username  |
+--------+-----------+
|      1 | username1 |
|      2 | user2     |
+--------+-----------+
2 rows in set (0.00 sec)
```

### 字段别名设置

select userid,username as un, address as addr from userinfo;

通过关键字as，将username设置为别名un，address设置为addr

这种别名设置是临时的，并不会改动原有字段名

```sh
mysql> select userid,username as un, address as addr from userinfo;
+--------+-----------+---------------+
| userid | un        | addr          |
+--------+-----------+---------------+
|      1 | username1 | guangzhou     |
|      2 | user2     | empty address |
+--------+-----------+---------------+
2 rows in set (0.00 sec)
```

### limit查询

**注意：mysql的位置是从0开始，与我们大部分编程语言中的索引是一样的**

**注意：mysql的位置是从0开始，与我们大部分编程语言中的索引是一样的**

**注意：mysql的位置是从0开始，与我们大部分编程语言中的索引是一样的**

limit是mysql中的一个特殊关键字，有三种使用方式

- limit 记录数, 从第一条开始查询 select * from userinfo limit 1;
- limit 起始位置，记录数  select * from userinfo limit 1,1;
- limit 记录数 offset 偏移 select * from userinfo limit 1 offset 1;

方式二和三的结果是一样的

```sh
mysql> select * from userinfo limit 2,3;
mysql> select * from userinfo limit 3 offset 2;

+--------+----------+----------+-----------+-------+---------------------+
| userid | username | password | address   | valid | birth               |
+--------+----------+----------+-----------+-------+---------------------+
|      3 | user3    | 123123   | guangzhou |     1 | 1989-01-01 01:01:01 |
|      4 | user4    | 123123   | guangzhou |     1 | 1989-01-01 01:01:01 |
|      5 | user5    | 123123   | guangzhou |     1 | 1989-01-01 01:01:01 |
+--------+----------+----------+-----------+-------+---------------------+
3 rows in set (0.00 sec)
```

### 条件查询

- and 并查询 select * from userinfo where username='user2' and password='123123';
- or 或查询 select * from userinfo where username='user2' or password='123123';
- between 区间查询 select * from userinfo where age between 30 and 35;
- in 子查询,只会查询in里面的条件，30岁和35岁，31-34不算在内 select * from userinfo where age in(30,35);
- is null 空查询 select * in userinfo where address is null;
- like 模糊查询 % 代表1个或者多个
  - select * from userinfo where username like '%us';  us在后面
  - select * from userinfo where username like 'us%';  us在前面
  - select * from userinfo where username like '%us%'; us在任何位置
  - select * from userinfo where username like '__us'; us前面必须有两个字符
  - select * from userinfo where username like binary '__Us'; 区分大小写

示例

```sh
mysql> select * from userinfo where username='user3' and password='123123';
+--------+----------+----------+-----------+-------+---------------------+
| userid | username | password | address   | valid | birth               |
+--------+----------+----------+-----------+-------+---------------------+
|      3 | user3    | 123123   | guangzhou |     1 | 1989-01-01 01:01:01 |
+--------+----------+----------+-----------+-------+---------------------+
1 row in set (0.00 sec)
```