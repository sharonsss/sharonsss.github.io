---
title: MySQL 的基本操作
tags: MySQL
categories: 数据库
index_img: /img/mysql.jpg
date: 2020-06-12 18:17:34
---


近期在维护的系统涉及到 `MySQL` 数据库的一些操作，都比较简单。没学习过数据库操作的知识，先在这里大致记录下用到的一些基本操作。

<!-- more -->

# 记录 MySQL 的操作

主要基于 `Terminal` 和 `Mysql Workbench` ，使用中涉及的一些步骤包括：

1. 登陆本地数据库、远程服务器数据库

2. 下载远程数据到本地

3. 将数据表导入数据库

4. 查看数据库中的数据

5. 向数据库中添加新数据

6. `Nginx` 图片文件夹添加对应的新图片（上传图片至服务器文件夹）

7. 重启系统，测试图片是否成功显示 - `YES`

8. 更改系统中 `html` 静态文件中的部分字段，将更改后的文件重新上传，并打包 `jar` 包

`6、7、8` 暂时不在这里记录。

# MySQL 数据库的基本操作

## 1. 在 `Terminal` 中登录

在 `Terminal` 中登录本地数据库，比较简单：

```
$ mysql -u 用户名 -p 密码
```

登录远程服务器：

```
$ mysql -u 远程服务器用户名 -p 远程服务器密码(具体密码可不填)  -h 远程服务器IP -P 远程服务器端口（一般为 3306）
$ Enter password:
```

密码输入正确的话，会出现如下的信息：

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is xxxx
Server version: 5.7.23 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```

说明成功连接到了远程服务器，接下来就可以进行一些基本操作了。

## 2. MySQL 的基本操作

想要查看系统所在的数据表，定位到具体的某个表格，这样操作（ MySQL 的语句后面要一定要加 `;` 符号）：

查看所有的数据库：

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| key_data           |
+--------------------+
2 rows in set (0.00 sec)
```

定位到一个具体的数据库中：

```
mysql> use key_data;
```

查看这个数据库中所有的数据表：

```
mysql> show tables;
+------------------------------+
| Tables_in_gain_external_data |
+------------------------------+
| table_1                      |
| table_2                      |
| table_3                      |
| table_4                      |
+------------------------------+
4 rows in set (0.00 sec)
```

查看某一个数据表的基本结构：

```
mysql> describe table_1;
+----------------------+---------------+------+-----+---------+-------+
| Field                | Type          | Null | Key | Default | Extra |
+----------------------+---------------+------+-----+---------+-------+
| id                   | int(11)       | NO   | PRI | 0       |       |
| chinese              | text          | NO   |     | NULL    |       |
| english              | text          | NO   |     | NULL    |       |
| classification1      | varchar(512)  | YES  |     | NULL    |       |
| classification2      | varchar(512)  | YES  |     | NULL    |       |
+----------------------+---------------+------+-----+---------+-------+
5 rows in set (0.01 sec)
```

预览数据表中前 `10` 行的数据：

```
mysql> SELECT * FROM key_data.table_1 LIMIT 0,10;
```

向数据表中添加数据，两种情况：

1. `id` 存在，只需要为已有 `id` 的数据添加一些已有字段的信息，使用 `update`；

2. `id` 不存在，需要完全添加新的数据，使用 `insert`。 如果表中还有其他字段，而且是必须填的，插入会出错，如果没有就成功了。

对于已有 `id` 的数据，举例操作如下：

```
mysql> update table_1 set image="xxx" where id=10;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

如果完全添加新数据，举例如下：

```
mysql> insert into 表名 （id,address） values (1,'柳州')
```

## 3. 导入导出数据表

这里我使用的是 **MySQL Workbench**，可以直观对 MySQL 数据库进行操作。

登录时连接项目所在的服务器地址，输入账号密码的登录之后，就可以进行操作了。

如果需要导入导出数据表，直接选择 `Table Data Import Wizard` 或者 `Table Data Export Wizard` 就可以。

<img src="/img/wizard.png" width="40%">

导出时，根据需要进行相关的设置，比如导出格式选择 `csv`、`JSON` 等等，分隔符、文本符号怎么显示等等。

![tu](/img/wizard2.png)

<br>

关于 Mysql 数据库的操作就记录到这里，如果后面遇到新操作，再来这里补充～

<br>

---

**参考来源：**
- *[MySql登陆服务器](https://blog.csdn.net/qq_41596568/article/details/88983561)*

- *[MySQL UPDATE：修改数据（更新数据）](http://c.biancheng.net/view/2579.html)*