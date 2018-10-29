# 数据接口

## 基础环境
- Cloudera 6.0.0
- Hadoop 3.0.0-cdh6.0.0
- Sqool 1.4.7-cdh6.0.0
- Hive 2.1.1-cdh6.0.0
- Hbase 2.0.0-cdh6.0.0

## 使用Sqool从关系型数据库导入数据到Hive和Hbase
### Sqool和Hive、Hbase简介
- Sqool
Sqoop是一个用来将Hadoop和关系型数据库中的数据相互转移的开源工具，可以将一个关系型数据库（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

- Hive
Hive起源于FaceBook，在Hadoop中扮演数据仓库的角色。建立在Hadoop集群的最顶层，对存储在Hadoop群上的数据提供类SQL的接口进行操作。你可以用 HiveQL进行select、join，等等操作。 
如果你有数据仓库的需求并且你擅长写SQL并且不想写MapReduce jobs就可以用Hive代替。
Hive的内置数据类型可以分为两大类：(1)、基础数据类型；(2)、复杂数据类型。其中，基础数据类型包括：TINYINT、SMALLINT、INT、BIGINT、BOOLEAN、FLOAT、DOUBLE、STRING、BINARY、TIMESTAMP、DECIMAL、CHAR、VARCHAR、DATE。 

- Hbase
HBase作为面向列的数据库运行在HDFS之上，HDFS缺乏随即读写操作，HBase正是为此而出现。HBase以Google BigTable为蓝本，以键值对的形式存储。项目的目标就是快速在主机内数十亿行数据中定位所需的数据并访问它。 
HBase是一个数据库，一个NoSql的数据库，像其他数据库一样提供随即读写功能，Hadoop不能满足实时需要，HBase正可以满足。如果你需要实时访问一些数据，就把它存入HBase。

- 在数据存储时可以用Hive作为静态数据仓库，HBase作为数据存储，放那些进行一些会改变的数据。在Hive中，普通表是存储在HDFS中，而你可以通过创建EXTERNAL TABLE外表来指定数据存储位置，可以是系统目录，也可以是ElasticSearch，还可以是HBase。 
在使用Sqoop从Mysql导出数据入Hadoop时，就需要考虑是直接入Hive（此时是普通表），还是导入数据到HBase，Sqoop同时支持导入这两种导入。

### 测试Sqoop
```shell
#测试数据库连接
[root@n1 ~]# sqoop list-databases --connect jdbc:mysql://10.66.27.22:3306/ --username root -P
.
.
.
18/10/18 15:14:36 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
information_schema
collect_platform
hive
metadata
monitor_platform
mysql
odps
scheduler

#测试SQL语句
[root@n1 ~]# sqoop eval \
--connect jdbc:mysql://10.66.27.22:3306/mysql \
--username root --password '123456' \
--query "SELECT Host, Db, User Select_priv \
FROM db"
.
.
.
18/10/18 15:13:41 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
------------------------------------------------------------------
| Host                 | Db                   | Select_priv      | 
------------------------------------------------------------------
| %                    | collect_platform     | collect_platform | 
| %                    | hive                 | hive_new_user    | 
| %                    | metadata             | metadata_user    | 
| %                    | monitor_platform     | monitor_platform | 
| %                    | odps                 | odps             | 
| %                    | scheduler            | scheduler        | 
| %                    | test                 |                  | 
| %                    | test\_%              |                  | 
------------------------------------------------------------------
```
经过以上测试可以确定Sqoop运行正常，连接正常。
###使用Sqoop从MySQL导入数据到Hive
HiveServer2 开始弃用了hive CLI而使用命令行程序Beeline。它是一种基于SQLline CLI 的JDBC客户端。
```shell
#打开beeline命令行
[root@n1 ~]# beeline
#用！connect命令连接hive数据库
beeline> !connect jdbc:hive2://n3:10000 scott tiger
#创建数据库
0: jdbc:hive2://n3:10000> create database testdb;
0: jdbc:hive2://n3:10000> show databases;
+----------------+
| database_name  |
+----------------+
| default        |
| testdb         |
+----------------+
2 rows selected (0.055 seconds)
#创建表格
0: jdbc:hive2://n3:10000> use testdb;
0: jdbc:hive2://n3:10000> create table testtable(
. . . . . . . . . . . . > Host string,
. . . . . . . . . . . . > Db string,
. . . . . . . . . . . . > Select_priv string)
. . . . . . . . . . . . > row format delimited fields terminated by ',';
0: jdbc:hive2://n3:10000>create table testtable(
Host string,
Db string,
Select_priv string)
row format delimited fields terminated by ',';

CREATE TABLE student (
  name string,
  sid int,
  majora string,
  tel string,
  birthday date
) row format delimited fields terminated by ',';


0: jdbc:hive2://n3:10000> show tables;
+------------+
|  tab_name  |
+------------+
| customer   |
| testtable  |
+------------+
2 rows selected (0.077 seconds)
#从mysql导入数据到hive
sqoop import \
--connect jdbc:mysql://10.66.27.22:3306/mysql \
--username root \
--password '123456' \
--table help_category \
-m 1 \
--hive-import \
--target-dir /user/hive/warehouse/testdb1 \
--hive-table  help_test \
--fields-terminated-by '\t' 

sqoop import --connect jdbc:mysql://10.66.27.22:3306/mysql --username root --password '123456'  --table help_category

sqoop import  --connect jdbc:mysql://10.66.27.22:3306/mysql --username root --password '123456' --table help_category --hive-import --fields-terminated-by ',' --hive-overwrite -m 1

```
