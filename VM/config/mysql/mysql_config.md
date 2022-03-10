# centOS 7 配置mysql
---
## 安装mysql
- 查看是否安装mysql
```
//若无显示，代表没用安装相关的mysql资源
[root@ecs-7 ~]# rpm -qa | grep mysql
```
- 查看是否含有mysql资源
```
[root@ecs-7 ~]# yum list | grep mysql
···
···
···
//会列出一堆mysql相关的资源，但是没有mysql-ccommunity-server的资源文件
//所以我们需要自己在mysql官网上下载对应资源
```
- 进入网址 https://dev.mysql.com/downloads/repo/yum/ 找到**Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package**下载源，点击下载可得到对应的文件**mysql80-community-release-el7-5.noarch.rpm** 
- 也可以使用命令行下载
```
//首先在~路径下创建一个文件夹存放即将下载的文件
[root@ecs-7 ~]# mkdir mysql

//然后进入新建的文件夹中
[root@ecs-7 ~]# cd mysql/

//下载资源
[root@ecs-7 ~]# wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

//安装
[root@ecs-7 ~]# yum -y install mysql57-community-release-el7-11.noarch.rpm

//查看，有显示代表安装成功
[root@ecs-7 ~]# rpm -qa | grep mysql
```
- 安装mysql-community-server
```
//安装，有选择就y
[root@ecs-7 ~]# yum install mysql-community-server --nogpgcheck
```
- 启动 mysql 服务
```
//启动服务
[root@ecs-7 ~]# systemctl start mysqld.service

//查看运行状态，running代表正在运行
[root@ecs-7 ~]# systemctl status mysqld
```
- 首次安装mysql的密码存放 /var/log/mysqld.log 文件中
```
//最后面的 vw(xtrf2e76F 就是密码
[root@ecs-7 ~]# grep "password" /var/log/mysqld.log
2022-03-09T05:06:09.098408Z 1 [Note] A temporary password is generated for root@localhost: vw(xtrf2e76F

//登录mysql，输入密码不可见
[root@ecs-7 ~]# mysql -u root -p
Enter password:

//修改密码，需要符合mysql的密码安全标准：密码长度与vw(xtrf2e76F一致，包括大小写字母，数字以及特殊字符
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY "your password";
Query OK, 0 rows affected (0.00 sec)    //没用显示ERROR代表成功

//退出，使用新密码即可登录
```
---
## 语法
### 基础语法
- 创建数据库---**CREATE**
``` SQL
CREATE DATABASE database_name;

--示例
mysql> CREATE DATABASE test;
Query OK, 1 row affected (0.00 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
| testSql            |
+--------------------+
```
- 使用数据库---**USE**
```SQL
USE database_name;
```
- 创建表---**CREATE TABLE**
```SQL
CREATE TABLE table_name(
    col_name type modifiers,
    ...,
    col_name type modifiers
);

--示例
mysql> CREATE TABLE mytable( id INT NOT NULL, str VARCHAR(20), PRIMARY kEY (id));
```
- 删除表---**DROP TABLE**
```SQL
DROP TABLE table_name;

--示例
mysql> DROP TABLE mytable;
```
- 删除数据库---**DROP**
``` SQL
DROP DATABASE database_name;

--示例
mysql> DROP DATABASE test;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testSql            |
+--------------------+
```
- 选择FORM中的列表并返回数据---**SELECT** and **FROM**
```SQL
-- 返回from_name中列表col_name_first , ... , col_name_last中的数据
SELECT col_name_first , ... , col_name_last FROM from_name;

-- 返回from_name中全部列表数据‘
SELECT * FROM from_name;

--示例1
mysql> SELECT name,sex FROM stu;
+------+-------+
| name | sex   |
+------+-------+
| yjl  | male  |
| mzh  | fmale |
| mery | fmale |
| mary | fmale |
| carh | fmale |
| Bob  | male  |
| kery | fmale |
| Kob  | male  |
+------+-------+

--示例2
mysql> SELECT * FROM stu;
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  0 |  19 | yjl  | male  |
|  1 |  18 | mzh  | fmale |
|  3 |  16 | mery | fmale |
|  4 |  18 | mary | fmale |
|  5 |  18 | carh | fmale |
|  6 |  21 | Bob  | male  |
|  7 |  17 | kery | fmale |
|  8 |  21 | Kob  | male  |
+----+-----+------+-------+
```
- 筛选---**WHERE**
```SQL
--组合用法
SELECT col_names
FROM from_name
WHERE conditions;

-- 示例1
mysql> SELECT * FROM stu WHERE sex = 'male';
+----+-----+------+------+
| id | age | name | sex  |
+----+-----+------+------+
|  0 |  19 | yjl  | male |
|  6 |  21 | Bob  | male |
|  8 |  21 | Kob  | male |
+----+-----+------+------+

-- 可以使用 AND OR NOT  =  !=(<>)  >  >=  < <=  等修饰
-- 示例2
mysql> SELECT * FROM stu WHERE sex = 'male' and id = 6;
+----+-----+------+------+
| id | age | name | sex  |
+----+-----+------+------+
|  6 |  21 | Bob  | male |
+----+-----+------+------+

-- 还有很多种修饰方法，之后讨论
```
- 限制搜索到的数据的显示条数以及规定数据的偏移量---**LIMIT** and **OFFSET**
```SQL
-- LIMIT 语句
SELECT col_names
FROM from_name
WHERE conditions -- 可有可无
LIMIT limit_number;

-- 示例1 显示stu table的前两行数据
mysql> SELECT * FROM stu LIMIT 2;
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  0 |  19 | yjl  | male  |
|  1 |  18 | mzh  | fmale |
+----+-----+------+-------+

-- OFSSET 语句
-- 语法1
SELECT col_names
FROM from_name
WHERE conditions -- 可有可无
LIMIT limit_number
OFFSET offset_number;
-- 语法2
SELECT col_names
FROM from_name
WHERE conditions -- 可有可无
LIMIT offset_number, limit_number;

-- 示例2 显示搜索到的数据偏移2个数据之后的两个数据
mysql> SELECT * FROM stu LIMIT 2 OFFSET 2;
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  3 |  16 | mery | fmale |
|  4 |  18 | mary | fmale |
+----+-----+------+-------+
```
- 排序---**ORDER BY**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE conditions -- 可有可无
ORDER BY col_names  -- 默认升序，降序在特定列名后加 DESC
LIMIT offset_number, limit_number;  -- 可有可无

-- 示例 将stu表种的数据按照age降序排序，然后返回偏移2个数据后的3个数据
mysql> SELECT * FROM stu ORDER BY age DESC LIMIT 2,3;
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  0 |  19 | yjl  | male  |
|  1 |  18 | mzh  | fmale |
|  4 |  18 | mary | fmale |
+----+-----+------+-------+
```
- 运算符---**IN**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE data IN (datas);

-- 示例
mysql> SELECT * FROM stu WHERE id IN (1,3,5);
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  1 |  18 | mzh  | fmale |
|  3 |  16 | mery | fmale |
|  5 |  18 | carh | fmale |
+----+-----+------+-------+
```
- 运算符---**BETWEEN AND**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE BETWEEN condition1 AND condition2;

-- 示例 显示age:17~19的数据
mysql> SELECT * FROM stu WHERE age BETWEEN 17 AND 19;
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  0 |  19 | yjl  | male  |
|  1 |  18 | mzh  | fmale |
|  4 |  18 | mary | fmale |
|  5 |  18 | carh | fmale |
|  7 |  17 | kery | fmale |
+----+-----+------+-------+
```

### 模糊查询与正则表达式

- 模糊搜索---**LIKE**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE col_name LIKE like_str;

-- 1. _ 表示单个字符
-- 2. % 表示一个字符串，字符串的长度可以为0

--示例1
mysql> SELECT * FROM stu WHERE name LIKE 'm__y';
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  3 |  16 | mery | fmale |
|  4 |  18 | mary | fmale |
+----+-----+------+-------+

--示例2
mysql> SELECT * FROM stu WHERE name LIKE '%r_';
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  3 |  16 | mery | fmale |
|  4 |  18 | mary | fmale |
|  5 |  18 | carh | fmale |
|  7 |  17 | kery | fmale |
+----+-----+------+-------+
```
- 正则表达式---**REGEXP**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE col_name REGEXP regexp_str;

-- 常用正则表达式
-- 1. ^ : 匹配输入字行首
-- 2. $ : 匹配输入行尾
-- 3. * : 匹配前面的子表达式任意次
-- 4. + : 匹配前面的子表达式一次或多次(大于等于1次）
-- 5. ? : 匹配前面的子表达式零次或一次
-- 6. x{n} x{n,} x{n,m} : 匹配确定的n(至少n)(至少n次至多m)次前面的子表达式x
-- 7. x|y : 匹配x或y
-- 8. [xyz] [^xyz] : 字符集合,匹配所包含(不包含)的任意一个字符
-- 9. [a-z] [^a-z] : 字符范围,匹配指定范围内(外)的任意字符
-- 10. ( ) : ()内的是一个子表达式
-- 11. | : 将两个匹配条件进行逻辑“或”（or）运算

-- 示例1
mysql> SELECT * FROM stu WHERE name REGEXP '^y';
+----+-----+------+------+
| id | age | name | sex  |
+----+-----+------+------+
|  0 |  19 | yjl  | male |
+----+-----+------+------+

-- 示例2
mysql> SELECT * FROM stu WHERE name REGEXP 'y$';
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  3 |  16 | mery | fmale |
|  4 |  18 | mary | fmale |
|  7 |  17 | kery | fmale |
+----+-----+------+-------+

-- 熟悉常用正则表达式即可
```
### 插入、更新和删除
- 插入数据---**INSERT INTO VALUES**
```SQL
-- 语法
INSERT INTO from_name VALUES (datas1),...,(datasn);

-- 示例 插入一个数据
mysql> INSERT INTO stu VALUES (2,12,'keria','male');
```
- 更新数据---**UPDATE SET WHERE**
```SQL
-- 语法
UPDATE from_name SET col_name WHERE conditions;

-- 示例 把name为keria的age更新为20
mysql> UPDATE stu SET age = 20 WHERE name = 'keria';
```
- 删除数据---**DELETE WHERE**


