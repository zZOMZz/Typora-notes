[toc]

# MySQL数据库

下载地址: [MySQL :: Download MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

文档: [MySQL :: MySQL 8.0 Reference Manual :: Preface and Legal Notices](https://dev.mysql.com/doc/refman/8.0/en/preface.html)

## 一. 数据库相关基础知识

### 1. 关系型数据库和非关系型数据库

- 关系型数据库: `MySQL, Oracle, DB2, SQL Server, Postgre SQL`
  - 关系型数据库我们通常会创建多个二维数据表
  - **数据表之间相互关联起来, 形成一对一, 一对多, 多对多的关系**
  - 之后可以利用SQL语句在**多张表**中查询我们所需的数据
- 非关系型数据库: `MongoDB , Redis, Memcached, HBse`等
  - 非关系型数据库(Not only SQL), 简称NoSQL
  - **非关系型数据库相对简单一些, 存储数据也会更加自由**(甚至可以直接将复杂的json对象直接塞入到数据库中)
  - NoSQL是**基于Key-Value的对应关系**, 并且**查询的过程中不需要经过SQL解析**
  - 可能存在冗余数据

- 两种数据库的选择
  - 后端开发时(Node, Java, Go等), 主要还是以**关系型数据库**为主
  - 比较常用的非关系型数据库在**爬取大量的数据进行存储**时, 比较常见



## 二. MySQL数据库基础知识

- MySQL是一个关系型数据库, 本质上就是一款软件, 一个程序
  - 这个程序管理多个数据库
  - 每个数据库中可以有多张表
  - 每个表中可以有多条数据

### 1. 连接MySQL

```shell
# 在自己的命令行下使用前需要先配置环境变量, 否则无法正确识别mysql命令
# 查看mysql的安装版本
mysql --version

# 连接mysql, username一般为root, password是自己在下载时设置的
mysql -u[username] -p[password]
# 连接上之后就能执行一些相关的SQL语句和数据库操作
```



### 2. MySQL默认数据库

![Snipaste_2023-01-12_13-36-49](.\图片\Snipaste_2023-01-12_13-36-49.png)

- `information_schema`: 信息数据库, 其中包括MySQL在维护的其他数据库, 表, 列, 访问权限等信息
- `performance_schema`: 性能数据库, 记录着MySQL Server数据库引擎运行过程中的一些资源消耗相关的信息
- `mysql`: 用于存储数据库管理者的用户信息, 权限信息以及一些日志信息等
- `sys`: 相当一是一个简洁版的performance_schema, 将性能数据库中的数据汇总成更容易理解的形式



### 3. 创建新数据库

- 简单演示

```shell
# 创建数据库
create database [name];
create database music_db;
# 进入创建的数据库
use music_db;
# 创建表
create table [name]
create table t_singer(name varchar(10),age int);
# 展示创建的表
show tables;
# 向表中插入数据
insert into t_singer(name,age) values ('五月天',18)
# 查询插入的数据
select * from t_singer
```



### 4. GUI工具介绍

- **Navicat**: 推荐但收费
- SQLYog
- TablePlus

