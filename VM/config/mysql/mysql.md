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

# 语法

## 数据定义语言---DDL

- 查看所有数据库名称
```SQL
-- 语法
SHOW DATABASE;

-- 示例
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

- 创建数据库
```SQL
-- 语法
CREATE DATABASE database_name;

-- 示例
mysql> CREATE DATABASE my_data_base;
```

- 切换数据库
```SQL
-- 语法
USE databse_name;

-- 示例
mysql> USE testSql;
Database changed    -- 代表成功
```

- 删除数据库
```SQL
-- 语法
DROP DATABASE database_name;

--示例
mysql> CREATE DATABASE my_data_base;
```

- 创建表
```SQL
-- 语法
CREATE TABLE table_name(
    col_name type modifiers,
    ...,
    col_name type modifiers
);

-- 示例
mysql> CREATE TABLE mytable( id INT NOT NULL, str VARCHAR(20), PRIMARY kEY (id));
```  

- 删除表
```SQL
-- 语法
DROP TABLE table_name;

-- 示例
mysql> DROP TABLE mytable;
```

- 显示所有表
```SQL
-- 语法
SHOW TABLES;

-- 示例
mysql> SHOW TABLES;
+-------------------+
| Tables_in_testSql |
+-------------------+
| stu               |
+-------------------+
```

- 查看表结构
```SQL
-- 语法
DESC table_name;

--示例
mysql> DESC stu;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| age   | int(11)     | NO   |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| sex   | char(5)     | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```

- 往表中添加列
```SQL
-- 语法
ALTER TABLE table_name ADD col_name type modifiers;

--示例
mysql> ALTER TABLE stu ADD test_col INT NOT NULL;
```

- 修改表中的列的类型
```SQL
-- 语法
ALTER TABLE table_name MODIFY col_name type modifiers;

--示例
mysql> ALTER TABLE stu MODIFY test_col CHAR(5) NOT NULL;
```

- 修改表中列的名称和列的类型
```SQL
-- 语法
ALTER TABLE table_name CHANGE col_name new_col_name type modifiers;

--示例
mysql> ALTER TABLE stu CHANGE test_col1 test_col2 INT NOT NULL;
```

- 删除表中的列
```SQL
-- 语法
ALTER TABLE table_name DROP col_name;

--示例
mysql> ALTER TABLE stu DROP test_col2;
```

- 修改表的名称
```SQL
-- 语法
ALTER TABLE table_name RENAME TO new_table_name;

--示例
mysql> ALTER TABLE stu RENAME TO student;
```


## 数据操作语言---DML

- 插入
```SQL
-- 语法1    插入完成数据
INSERT INTO from_name VALUES (datas1),...,(datasn);

-- 语法2    给定列，插入对应列的数据
INSERT INTO from_name(col_name,...,col_name) VALUES (datas),...,(datas);

-- 示例1 插入一个数据
mysql> INSERT INTO stu VALUES (2,12,'keria','male');

-- 示例2 插入一个数据
mysql> INSERT INTO stu(id,name) VALUES (2,'keria');
```

- 更新
```SQL
-- 语法     更新符合条件的列
UPDATE from_name SET col_name = up_date WHERE conditions;

-- 示例
mysql> UPDATE stu SET age = 20 WHERE name = 'keria';
```

- 删除
```SQL
-- 语法1    删除表，这种删除可以回滚
DELETE FROM from_name;

-- 语法2    删除符合条件的行
DELETE FROM from_name WHERE conditions;

-- 示例1
mysql> DELETE FROM stu;

-- 示例2
mysql> DELETE FROM stu WHERE id = 1;
```


## 数据查询语言---DQL

- 查询数据
```SQL
-- 语法1    返回查询列的数据
SELECT col_name_first , ... , col_name_last 
FROM from_name;

-- 语法2    返回所有列的数据
SELECT * 
FROM from_name;

-- 示例1
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

-- 示例2
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

- 条件筛选
```SQL
-- 语法 返回符合条件的对应列的数据
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

-- 示例2
mysql> SELECT * FROM stu WHERE sex = 'male' AND id = 6;
+----+-----+------+------+
| id | age | name | sex  |
+----+-----+------+------+
|  6 |  21 | Bob  | male |
+----+-----+------+------+
```

- 限制数据的显示条数和偏移量
```SQL
-- LIMIT 语句
-- 语法
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

- 排序
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

### 条件查询

- **IN(set)**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE data IN (set);

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

- **BETWEEN ... AND ...**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE BETWEEN ... AND ... ;

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

- **AND**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE  ... AND ... ;

-- 示例 返回id>5并且id<10的数据
mysql> SELECT * FROM stu WHERE id > 5 AND id < 10;
```

- **OR**
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE  ... OR ... ;

-- 示例 返回id<5或者id>10的数据
mysql> SELECT * FROM stu WHERE id < 5 OR id > 10;
```

- **NOT**
```SQL
-- NOT 是非的意思 即取所修饰表达式对立的一面
-- 语法
SELECT col_names
FROM from_name
WHERE NOT ... ;

-- 示例 返回id>5并且id<10的数据
mysql> SELECT * FROM stu WHERE NOT (id < 5 OR id > 10);
```

## 模糊查询与正则表达式

- 模糊搜索
```SQL
-- 语法
SELECT col_names
FROM from_name
WHERE col_name LIKE like_str;

-- 1. _ 表示单个字符
-- 2. % 表示一个字符串，字符串的长度可以为0

-- 示例1
mysql> SELECT * FROM stu WHERE name LIKE 'm__y';
+----+-----+------+-------+
| id | age | name | sex   |
+----+-----+------+-------+
|  3 |  16 | mery | fmale |
|  4 |  18 | mary | fmale |
+----+-----+------+-------+

-- 示例2
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
- 正则表达式
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

<!-- ### 数据控制语言---DCL -->


