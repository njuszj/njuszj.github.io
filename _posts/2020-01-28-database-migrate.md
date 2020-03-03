---
layout: post
title:  数据库迁移:从mysql到postgres
date:   2020-01-28 19:55:00 +0800
categories: note
tag: 项目
---

* content
{:toc}



现在由于某些原因我需要把一个MySQL数据库中的数据迁移到PostgreSQL中，本来打算使用python脚本解决，后来通过简单的导出导入sql文件解决了迁移问题。

### MySQL端: 导出.sql备份文件
使用mysqldump命令导出完整的表结构和表数据
`mysqldump dbname -u username -p --tables table_name > /.../.../xxx.sql`
如只需导出表结构，在dbname前加上-d参数

### PostgreSQL端: 导入.sql文件
`psql -d dbname -h 127.0.0.1 -p 5432 -U username -f /.../.../xxx.sql`

注意，直接这样执行肯定是不会成功的，因为mysql的一些命令是无法在postgresql中执行的，所以会报错。现在要做的就是对第一步从mysql中导出的.sql文件进行一定的修改，比如:
+ 删除所有 ` 字符
+ 注释掉`ENGINE=InnoDB DEFAULT CHARSET=utf8;`
+ 注释掉`LOCK TABLES goodsInfo WRITE;`
+ 注释掉`UNLOCK TABLES;`
+ 一些数据类型的差异，比如日期(datetime-timestamp)
+ ...

不确定的话就一直尝试直到没有报错，但是这里面有一个地方比较坑的是postgres里面在9.0版本后修改了转移方式，将反斜杠的转义功能去除了，如果sql语句用到了反斜杠转义就需要在postgres的控制台修改一个变量使反斜杠重新变为转义字符。
`sudo -u postgres psql`
`SET standard_conforming_strings = off;`

我在上述步骤完成后成功的迁移了数据。
