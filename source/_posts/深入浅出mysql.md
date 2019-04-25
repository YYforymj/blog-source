---
title: 深入浅出mysql
date: 2019-04-25 10:23:17
categories: 数据库
---

> 以下为深入浅出mysql一书的读书笔记

<!-- more -->

 1. 每个MyISAM在磁盘上存储成三个文件。文件名都和表名相同，扩展名分别是.frm （存储表定义）、 .MYD (MYData，存储数据)、 .MYI (MYIndex，存储索引)。数据文件和索引文件可以放置在不同的目录，平均分布io，获得更快的速度。

2. InnoDB存储引擎提供了具有提交、 回滚和崩溃恢复能力的事务安全。 但是对比Myisam的存储引擎， InnoDB写的处理效率差一些并且会占用更多的磁盘空间以保留数据和索引。

3. varchar数据类型

   对于InnoDB数据表，内部的行存储格式没有区分固定长度和可变长度列（所有数据行都使用指向数据列值的头指针） ，因此在本质上，使用固定长度的 CHAR列不一定比使用可变长度VARCHAR列简单。 因而， 主要的性能因素是数据行使用的存储总量。 由于CHAR平均占用的空间多于VARCHAR， 因 此使用VARCHAR来最小化需要处理的数据行的存储总量和磁盘I/O是比较好的。

4. 字符集：Mysql的字符集包括字符集（ CHARACTER）和校对规则（ COLLATION）两个概念。字符集是用来定义mysql存储字符串的方式，校对规则则是定义了比较字符串的方式。字符集和校对规则有4个级别的默认设置：服务器级、数据库级、表级和字段级。分别在不同的地方设置，作用也不相同。

5. 索引的设计与使用

   - 使用短索引。 如果对串列进行索引， 应该指定一个前缀长度， 只要有可能就应该这样做。例如，如果有一个CHAR(200) 列，如果在前10 个或20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。对前10 个或20 个字符进行索引能够节省大量索引空间，也可能会使查询更快。较小的索引涉及的磁盘I/O 较少，较短的值比较起来更快。更为重要的是， 对于较短的键值， 索引高速缓存中的块能容纳更多的键值， 因此， MySQL也可以在内存中容纳更多的值。这增加 了找到行而不用读取索引中较多块的可能性。（当然， 应该利用一些常识。 如仅用列值的第一个字符进行索引是不可能有多大好处的，因为这个索引中不会有许多不 同的值。 

   - 使用惟一索引。考虑某列中值的分布。对于惟一值的列，索引的效果最好，而具有多个重复值的列，其索引效果最差。

   -  用最左前缀。在创建一个n 列的索引时，实际是创建了MySQL可利用的n 个索引。多列索引可起几个索引的作用， 因为可利用索引中最左边的列集来匹配行。 这样的列集称为最左前缀。 

6. 事务控制

   - MySQL通过SET AUTOCOMMIT, START TRANSACTION, COMMIT和ROLLBACK等语句支持本地事务。语法：

     ```
   START TRANSACTION | BEGIN [WORK]
     COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
     ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
     SET AUTOCOMMIT = {0 | 1}
     ```
     
   - 默认情况下， mysql是autocommit的，如果需要通过明确的commit和rollback来提交和回滚事务， 那么需要通过明确的事务控制命令来开始事务。

     同一个事务中，最好不使用不同存储引擎的表，否则rollback时需要对非事务类型的表进行特别的处理，因为commit、 rollback只能对事务类型的表进行提交和回滚。通常情况下， 只对提交的事务纪录到二进制的日志中， 但是如果一个事务中包含非事务类型的表， 那么回滚操作也会被记录到二进制日志中， 以确保非事务类型表的更新可以被复制到从的数据库中 。

     事务中可以通过定义savepoint，指定回滚事务的一个部分，但是不能指定提交事务的一个部分。对于复杂的应用，可以定义多个不同的savepoint，满足不同的条件时，回滚不同的savepoint。需要注意的是，如果定义了相同名字的savepoint，则后面定义的savepoint会覆盖之前的定义。 对于不再需要使用的savepoint， 可以通过release savepoint命令删除savepoint，删除后的savepoint，不能再执行rollback to savepoint命令。

   - 注意问题：在MySQL中， 数据库对应数据目录中的目录。 数据库中的每个表至少对应数据库目录中的一个文件(也可能是多个，取决于存储引擎)。因此，所使用操作系统的大小写敏感性决定了数据库名和表名的大小写敏感性。如果你正使用InnoDB表，在任何平台上均应将lower_case_tables_name设置为1，以强制将名转换为小写。

7. sql优化

   - SHOW STATUS可以提供服务器状态信息。以下几个参数对Myisam和Innodb存储引擎都计数：

     - Com_select 执行select操作的次数，一次查询只累加1；

     - Com_insert 执行insert操作的次数， 对于批量插入的insert操作， 只累加一次；

     - Com_update 执行update操作的次数；

     - Com_delete 执行delete操作的次数；

   - 以下几个参数是针对Innodb存储引擎计数的，累加的算法也略有不同：

     - Innodb_rows_read select查询返回的行数；

     - Innodb_rows_inserted执行Insert操作插入的行数；

     - Innodb_rows_updated 执行update操作更新的行数；

     - Innodb_rows_deleted 执行delete操作删除的行数；

     通过以上几个参数， 可以很容易的了解当前数据库的应用是以插入更新为主还是以查询操作为主， 以及各种类型的SQL大致的执行比例是多少。 对于更新操作的计数，是对执行次数的计数，不论提交还是回滚都会累加。

     对于事务型的应用， 通过Com_commit和Com_rollback可以了解事务提交和回滚的情况，对于回滚操作非常频繁的数据库，可能意味着应用编写存在问题。

8. 可以通过以下两种方式定位执行效率较低的SQL语句。

   - 可以通过慢查询日志定位那些执行效率较低的sql语句，用--log-slowqueries[=file_name]选项启动时， mysqld写一个包含所有执行时间超过long_query_time秒的SQL语句的日志文件。可以链接到管理维护中的相关章节。

   - 慢查询日志在查询结束以后才纪录， 所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用show processlist命令查看当前MySQL在进行的线程，包括线程的状态，是否锁表等等，可以实时的查看SQL执行情况， 同时对一些锁表操作进行优化。

   - 一些情况下， 子查询可以被更有效率的连接(JOIN)替代。

9. 假设我们要将所有没有订单记录的用户取出来，可以用下面这个查询完成：

   ```
   SELECT * FROM customerinfo WHERE CustomerID NOT in (SELECT CustomerID FROM salesinfo )
   ```

   如果使用连接(JOIN).. 来完成这个查询工作， 速度将会快很多。 尤其是当salesinfo 表中对CustomerID建有索引的话，性能将会更好，查询如下：

   ```
   SELECT * FROM customerinfo LEFT JOIN salesinfo ON customerinfo.CustomerID=salesinfo.CustomerID WHERE salesinfo.CustomerID IS NULL
   ```

   连接(JOIN).. 之所以更有效率一些，是因为 MySQL不需要在内存中创建临时表。

10. 负载均衡

    mysql的主从复制可以有效的分流更新操作和查询操作，具体的实现是一个主服务器，承担更新操作，多台从服务器，承担查询操作，主从之间通过复制实现数据的同步。多台从服务器一方面用来确保可用性，一方面可以创建不同的索引满足不同查询的需要。

11. binlog

    记录内容：二进制日志包含了所有更新了数据或者已经潜在更新了数据（例如，没有匹配任何行的一个DELETE）的所有语句。语句以“ 事件” 的形式保存，它描述数据更改文件位置和格式：当用--log-bin[=file_name]选项启动时， mysqld写入包含所有更新数据的SQL命令的日志文件。如果未给出file_name值， 默认名为-bin后面所跟的主机名。如果给出了文件名，但没有包含路径，则文件被写入数据目录。

​    