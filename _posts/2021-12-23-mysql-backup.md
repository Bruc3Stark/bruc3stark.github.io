---
layout: mypost
title: MySQL备份命令
categories: [数据库]
---

最近服务器到期要迁移数据，发现使用 Navicat 导出数据太慢了，这里学习下 mysqldump 的使用

### 参数

命令行格式如下

```
[root@VM_224_232_centos ~]# mysqldump
Usage: mysqldump [OPTIONS] database [tables]
OR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
OR     mysqldump [OPTIONS] --all-databases [OPTIONS]
For more options, use mysqldump --help
```

这里参数有点多，只需要了解下常用的就行了，所有参数可以参考[官方文档](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)

注意参数之间的空格，有些参数和值是紧挨着的，没有空格

```
# 默认localhost
-h[主机名]

# 大写的P，不指定默认3306，同时指定了-h后-P才会使用
-P[端口]

-u[用户名]
-p[密码]

# 一般是第一个名字参数作为数据库名，后面的作为表名，使用该项后都是数据库名，无法指定单个表
# 导出多个数据库，不做限定的话是备份表结构和表数据
--database [db1 db2]

# 仅导出表结构
-d

# 仅导出表数据
-t
```

### 导出

注意：`[Warning] Using a password on the command line interface can be insecure.`忽略就行了，意思是你在命令行中输入了密码不安全

```
# 导出cooking数据库
mysqldump -uroot -p123456789 cooking > cooking_all.sql
# 导出cooking数据库建表语句
mysqldump -uroot -p123456789 cooking -d > cooking_create.sql
# 导出cooking数据库插入语句
mysqldump -uroot -p123456789 cooking -t > cooking_data.sql
# 导出cooking数据库中的bc_cook_wx_user表
mysqldump -uroot -p123456789 cooking bc_cook_wx_user > bc_cook_wx_user.sql
```

### 还原

这里使用 mysqladmin 命令，直接一条命令执行一条 sql，就是非交互式的执行 sql 命令

当然了也可以登陆进去,使用`source xxx.sql`来执行脚本

```
# 先创建数据库
mysqladmin -uroot -p123456789 create cooking
# 指定在那个数据库执行脚本
mysql -uroot -p123456789 cooking  < cooking_create.sql
```
