# centOS 7 配置mysql

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
Enter password: input your password

//修改密码，需要符合mysql的密码安全标准：密码长度与vw(xtrf2e76F一致，包括大小写字母，数字以及特殊字符
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY "your password";
Query OK, 0 rows affected (0.00 sec)    //没用显示ERROR代表成功

//退出，使用新密码即可登录
```
---

# 运程链接mysql

- 本节知识会用到DCL命令
```SQL
-- 1. 登录mysql
[root@ecs-7 ~]# mysql -u root -p
Enter password: input your password

-- 2. 查看数据库
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

-- 3. 选择mysql数据库
mysql> USE mysql;

-- 4. 创建一个新用户 user_name:用户名称  domain_name:域名或者ip
CREATE USER 'user_name'@'domain_name' IDENTIFIED BY 'password';
--示例 表示创建admin用户，%代表可以在任何主机登录此服务器的mysql，其密码为123456
mysql> CREATE USER 'admin'@'%' IDENTIFIED BY '123456';

-- 5. 查看用户是否创建成功
mysql> SELECT Host, User FROM mysql;
+------+----------+
| Host | User     |
+------+----------+
| %    | admin    |
+------+----------+

-- 6. 查看用户权限 *.*代表用户只有登录权限
mysql> SHOW GRANTS FOR 'admin'@'%';
+--------------------------------------+
| Grants for mushroom@%                |
+--------------------------------------+
| GRANT USAGE ON *.* TO 'admin'@'%'    |
+--------------------------------------+

-- 7. 赋予用户超级权限
mysql> GRANT ALL ON *.* TO 'admin'@'%' WITH GRANT OPTION;
```

- 经过以上操作即可在任何主机上使用**admin**用户登录你在服务器上的部署的mysql
- 由于mysql需要使用到3306端口，查看服务器3306端口的开启情况

```
1. 查看服务器端口占用情况
[root@ecs-7 ~]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:39946         0.0.0.0:*               LISTEN      16691/node          
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      7738/nginx: master  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1291/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      895/master          
tcp6       0      0 :::3306                 :::*                    LISTEN      910/mysqld          
tcp6       0      0 :::80                   :::*                    LISTEN      7738/nginx: master  
tcp6       0      0 :::22                   :::*                    LISTEN      1291/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      895/master          

3306端口是mysql使用的端口

2.查看防火墙是否开启了3306端口
[root@ecs-7 ~]# firewall-cmd --query-port=3306/tcp
no

表示，防火墙没用开启3306端口

3.永久开启3306端口
[root@ecs-7 ~]# firewall-cmd --permanent --add-port=3306/tcp
success

success代表成功

4.重启防火墙
[root@ecs-7 ~]# firewall-cmd --reload

5.再次查询3306端口
[root@ecs-7 ~]# firewall-cmd --query-port=3306/tcp
yes

yes代表3306已经开启
```

- 有时候即使防火墙开启了3306端口，其他主机也访问不了mysql，这是由于服务器安全组没用允许3306端口通信，这个时候需要到所购买服务器的控制台，找到安全组选项，往安全组里添加新的出入规则即可（即在安全组里允许3306端口开启）

- **vscode连接mysql**

  1. 在拓展里安装 MySQL 与 MySQL Syntax,然后就可以在vscode资源管理器中看到MYSQL资源；
  2. 点击MYSQL资源管理器右边的+号，可以创建一个mysql连接，依次输入 host=>user=>password=>port=>空输入 ;
  3. 展开MYSQL资源管理器会看到你已经创建过的mysql连接
  4. 连接成功后，即可使用

---

# 基本语法

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

- 插入数据
```SQL
-- 使用 DEFAULT   使某列插入默认值
-- 语法1    插入完成数据
INSERT INTO from_name VALUES (datas1),...,(datasn);

-- 语法2    给定列，插入对应列的数据
INSERT INTO from_name(col_name,...,col_name) VALUES (datas),...,(datas);

-- 语法3    从其他表中插入数据
INSERT INTO from_name(col_name,...,col_name)    -- 两表的列名要相同
SELECT (col_name,...,col_name)
FROM another_from_name;

-- 示例1 插入一个数据
mysql> INSERT INTO stu VALUES (2,12,'keria','male');

-- 示例2 插入一个数据
mysql> INSERT INTO stu(id,name) VALUES (2,'keria');
```

- 复制表
```SQL
-- 语法 将from_name表中的数据复制到新创的table_name中
--      缺点就是这样复制有缺陷，没有设置列的属性，比如主键没有设置
CREATE TABLE table_name AS
SELECT col_names
FROM from_name;
```

- 更新数据
```SQL
-- 语法     更新符合条件的列
UPDATE from_name SET col_name = up_date WHERE conditions;

-- 示例
mysql> UPDATE stu SET age = 20 WHERE name = 'keria';
```

- 删除数据
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
SELECT col_name , ... , col_name 
FROM from_name;

-- 语法2    返回所有列的数据
SELECT * 
FROM from_name;

-- 语法3    使用完成限定的表名
SELECT from_name.col_name , ... ,from_name.col_name 
FROM from_name;  -- from_name 也可用 database_name.from_name 

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

-- 示例3
mysql> SELECT stu.id , stu.name FROM stu;
+----+-------+
| id | name  |
+----+-------+
|  0 | yjl   |
|  1 | mzh   |
|  3 | mery  |
|  4 | mary  |
|  5 | carh  |
|  6 | Bob   |
|  7 | kery  |
|  8 | Kob   |
+----+-------+
```

- **DISTINCT**
```SQL
-- 使用关键字 DISTINCT 修饰列使该列只返回不同的数据（即同名数据只显示一次）
-- 语法1    返回查询列的数据且只返回不同的数据
SELECT DISTINCT col_name_first , ... , DISTINCT col_name_last 
FROM from_name;

-- 示例 未使用DISTINCT修饰，明显看到有相同行的数据
mysql> SELECT age ,sex FROM stu;
+-----+-------+
| age | sex   |
+-----+-------+
|  19 | male  |
|  18 | fmale |
|  20 | male  |
|  16 | fmale |
|  18 | fmale |
|  18 | fmale |
|  21 | male  |
|  17 | fmale |
|  21 | male  |
+-----+-------+

-- 示例 使用DISTINCT修饰，只返回唯一数据
mysql> SELECT DISTINCT age ,sex FROM stu;
+-----+-------+
| age | sex   |
+-----+-------+
|  19 | male  |
|  18 | fmale |
|  20 | male  |
|  16 | fmale |
|  21 | male  |
|  17 | fmale |
+-----+-------+

-- 注意：不能部分使用 DISTINCT 因为该关键字应用于所有列而不仅是前置它的列
--       例如：SELECT DISTINCT col_name1 , col_name2 ... 这里DISTINCT
--       修饰 col_name1 , col_name2 ，即返回的每行数据是唯一的
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

- 分组---**GROUP BY**
```SQL
-- 对查询返回的数据根据某个字段进行分组
-- 语法
SELECT col_names
FROM from_name
WHERE conditions        -- 筛选未分组前符合条件的数据
GROUP BY col_names      -- 对数据进行分组
ORDER BY col_names;     -- 对数据进行排序    

-- 示例1 查看customer表中具有相同status_id的个数
mysql> SELECT status_id,COUNT(id) as count
    -> FROM customer
    -> GROUP BY status_id;
+-----------+-------+
| status_id | count |
+-----------+-------+
|      NULL |     3 |   -- NULL 有三个
|         1 |     4 |   -- 1 有4个
|         2 |     2 |
|         3 |     2 |
+-----------+-------+

-- 示例2 内连接customer和order_status找出各状态的总人数
mysql> SELECT os.name as status,
    ->        COUNT(c.id) as count
    -> FROM customer c
    -> INNER JOIN order_status os
    ->     USING(status_id)
    -> GROUP BY os.name;
+-----------+-------+
| status    | count |
+-----------+-------+
| delivered |     2 |
| processed |     4 |
| shipped   |     2 |
+-----------+-------+
```

- 子句---**HAVING**
```SQL
-- HAVING 对分组后的数据进行筛选
-- 语法
SELECT col_names
FROM from_name
WHERE conditions        -- 筛选未分组前符合条件的数据
GROUP BY col_names      -- 对数据进行分组
HAVING conditions;      -- 对分组后的数据进行筛选
ORDER BY col_names      -- 对数据进行排序

-- 示例 对两张表进行内连接，然后以status_id进行分组，并且分组后进行数据筛选
--      去掉status为NULl的记录，然后对记录根据count进行降序排序
mysql> SELECT os.name as status,
    ->        COUNT(c.id) as count
    -> FROM customer c
    -> LEFT OUTER JOIN order_status os
    ->     USING(status_id)
    -> GROUP BY os.name
    -> HAVING NOT status IS NULL
    -> ORDER BY count DESC;
+-----------+-------+
| status    | count |
+-----------+-------+
| processed |     4 |
| delivered |     2 |
| shipped   |     2 |
+-----------+-------+
```
- 汇总数据---**WITH ROllUP**
```SQL
-- 注意：使用 WITH ROLLUP，此函数是对聚合函数进行求和
--       注意 with rollup是对 group by 后的第一个字段，进行分组求和
-- 语法
SELECT col_names
FROM from_name
WHERE conditions        -- 筛选未分组前符合条件的数据
GROUP BY col_names      -- 对数据进行分组
WITH ROLLUP;            -- 对聚合函数进行求和

-- 示例 对count字段的数据进行汇总
mysql> SELECT os.name as status,
    ->        COUNT(c.id) as count
    -> FROM customer c
    -> LEFT OUTER JOIN order_status os
    ->     USING(status_id)
    -> GROUP BY os.name WITH ROLLUP;
+-----------+-------+
| status    | count |
+-----------+-------+
| NULL      |     3 |
| delivered |     2 |
| processed |     4 |
| shipped   |     2 |
| NULL      |    11 |
+-----------+-------+
```

- ALL关键字
```SQL
-- 将单个值与子查询返回的单列值集进行比较，必须全部满足才行
SELECT col_names
FROM from_name
WHERE col_name [> >= <> = < <=] ALL ( 子查询 );

-- 示例
mysql> SELECT *
    -> FROM customer
    -> WHERE age > ALL(
    ->     SELECT DISTINCT age
    ->     FROM customer
    ->     WHERE sex = 'fmale' && age<=20
    -> );
+----+-----+--------+-------+-----------+
| id | age | name   | sex   | status_id |
+----+-----+--------+-------+-----------+
|  0 |  19 | yjl    | male  |         2 |
|  2 |  20 | keria  | male  |      NULL |
|  6 |  21 | Bob    | male  |         1 |
|  8 |  21 | Kob    | male  |         3 |
|  9 |  19 | Gropry | male  |         3 |
| 10 |  21 | Vae    | fmale |         2 |
+----+-----+--------+-------+-----------+
```

- ANY关键字
```SQL
-- 将单个值与子查询返回的单列值集进行比较，有一个满足即可
SELECT col_names
FROM from_name
WHERE col_name [> >= <> = < <=] ANY ( 子查询 );

-- 示例
mysql> SELECT *
    -> FROM customer
    -> WHERE status_id = ANY(
    ->     SELECT DISTINCT status_id
    ->     FROM order_status
    -> );
+----+-----+--------+-------+-----------+
| id | age | name   | sex   | status_id |
+----+-----+--------+-------+-----------+
|  0 |  19 | yjl    | male  |         2 |
|  1 |  18 | mzh    | fmale |         1 |
|  3 |  16 | mery   | fmale |         1 |
|  5 |  18 | carh   | fmale |         1 |
|  6 |  21 | Bob    | male  |         1 |
|  8 |  21 | Kob    | male  |         3 |
|  9 |  19 | Gropry | male  |         3 |
| 10 |  21 | Vae    | fmale |         2 |
+----+-----+--------+-------+-----------+
```

- EXISTS关键字
```SQL
-- EXISTS 是对外表作loop循环,即对外表的每一行都对内表进行比较
--        EXISTS用于检查子查询是否至少会返回一行数据，该子查询
--        实际上并不返回任何数据，而是返回值True或False
--        EXISTS 指定一个子查询，检测行的存在
-- 语法
SELECT col_names
FROM from_name
WHERE EXISTS ( 子查询 );

-- 示例 只要子查询能够查到数据就能返回TRUE
mysql> SELECT * 
    -> FROM customer c
    -> WHERE EXISTS (
    ->     SELECT status_id
    ->     FROM order_status
    ->     WHERE c.status_id=status_id
    -> );
+----+-----+--------+-------+-----------+
| id | age | name   | sex   | status_id |
+----+-----+--------+-------+-----------+
|  0 |  19 | yjl    | male  |         2 |
|  1 |  18 | mzh    | fmale |         1 |
|  3 |  16 | mery   | fmale |         1 |
|  5 |  18 | carh   | fmale |         1 |
|  6 |  21 | Bob    | male  |         1 |
|  8 |  21 | Kob    | male  |         3 |
|  9 |  19 | Gropry | male  |         3 |
| 10 |  21 | Vae    | fmale |         2 |
+----+-----+--------+-------+-----------+
```
- SELECT语句中的子查询
```SQL
-- 一般对聚合函数进行子查询作为新的一列值
-- 语法
SELECT col_names,
       (子查询:相当于返回一列值),
FROM from_name;
-- 注意：计算列表达式中不能使用列的别名，要使用真实列名
--       假如要使用列的别名可以这样使用(SELECT alias)

-- 示例 这样聚合函数AVG(age)自成一列
mysql> SELECT * ,
    ->     (SELECT AVG(age) FROM customer) AS avg_of_age
    -> FROM customer;
+----+-----+--------+-------+-----------+------------+
| id | age | name   | sex   | status_id | avg_of_age |
+----+-----+--------+-------+-----------+------------+
|  0 |  19 | yjl    | male  |         2 |    18.9091 |
|  1 |  18 | mzh    | fmale |         1 |    18.9091 |
|  2 |  20 | keria  | male  |      NULL |    18.9091 |
|  3 |  16 | mery   | fmale |         1 |    18.9091 |
|  4 |  18 | mary   | fmale |      NULL |    18.9091 |
|  5 |  18 | carh   | fmale |         1 |    18.9091 |
|  6 |  21 | Bob    | male  |         1 |    18.9091 |
|  7 |  17 | kery   | fmale |      NULL |    18.9091 |
|  8 |  21 | Kob    | male  |         3 |    18.9091 |
|  9 |  19 | Gropry | male  |         3 |    18.9091 |
| 10 |  21 | Vae    | fmale |         2 |    18.9091 |
+----+-----+--------+-------+-----------+------------+
```

- FROM语句子查询
```SQL
-- 将子查询返回的表作为本次查询的表
-- 语法
SELECT col_names
FROM (子查询:返回表) AS alias   -- 此处的alias必须有
...;    -- 其他子语句

-- 示例 将子查询返回的表作为新表并返回新表中id<5的记录
mysql> SELECT *
    -> FROM (
    ->     SELECT *
    ->     FROM family
    ->     WHERE sex = 'fmale'
    -> ) AS new_from
    -> WHERE id<5;
+----+------+-------+-----------+-----------+
| id | name | sex   | father_id | mother_id |
+----+------+-------+-----------+-----------+
|  2 | mery | fmale |      NULL |      NULL |
|  4 | pou  | fmale |         1 |         2 |
+----+------+-------+-----------+-----------+
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

-- 1. _ 表示任意单个字符
-- 2. % 表示任何字符出现任意次数，次数可以为0

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

-- 注意：不要过于过度依赖通配符，它会减慢搜索效率
--       除非必要，否则不要将通配符放在搜索处的开头，这将使搜索效率最慢
```
- 正则表达式
```SQL
--注意：mysql的正则表达式仅仅支持完整正则表达式的一个子集

-- BINARY:mysql正则表达式不区分大小写，使用BINARY修饰REGEXP可区分大小
-- 用法 REGEXP BINARY

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
-- 12. . : 匹配任意一个字符

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
---
# 函数
## 文本处理函数
- 拼接串函数---**Concat()**
```SQL
-- 语法
Concat(str_1,...,str_n)  --str_i可以使列名也可以是字符串

-- 示例
mysql> SELECT Concat('name:',name,', age',age) FROM stu;
+----------------------------------+
| Concat('name:',name,', age',age) |
+----------------------------------+
| name:yjl, age19                  |
| name:mzh, age18                  |
| name:keria, age20                |
| name:mery, age16                 |
| name:mary, age18                 |
| name:carh, age18                 |
| name:Bob, age21                  |
| name:kery, age17                 |
| name:Kob, age21                  |
+----------------------------------+

-- 觉得列名太丑？可以给列名起一个别名
```
- 给列起别名---**AS**
```SQL
-- 语法
SELECT col_name AS alias
FROM from_name;

-- 示例
mysql> SELECT Concat('name:',name,', age',age) AS info FROM stu;
+-------------------+
| info              |
+-------------------+
| name:yjl, age19   |
| name:mzh, age18   |
| name:keria, age20 |
| name:mery, age16  |
| name:mary, age18  |
| name:carh, age18  |
| name:Bob, age21   |
| name:kery, age17  |
| name:Kob, age21   |
+-------------------+
```
- 给算数运算的列起别名
```SQL
-- 语法
SELECT 列的算数计数 AS alias
FROM from_name;

-- 示例
mysql> SELECT name,price,number,price*number AS total_price FROM myorder;
+----------+-------+--------+--------------------+
| name     | price | number | total_price        |
+----------+-------+--------+--------------------+
| pen      |   1.2 |    100 | 120.00000476837158 |      -- 此处我有疑问...
| notebook |   3.5 |     50 |                175 |
| rule     |     2 |     85 |                170 |
| bag      |    15 |     35 |                525 |
+----------+-------+--------+--------------------+

```
- 去空格函数---**Trim()**
```SQL
-- 语法
Trim(str);       --去掉str字符串左右两边的空格
RTrim(str);      --去掉str字符串右边的空格
LTrim(str);      --去掉str字符串左边的空格
```
- 等以后再讨论

## 聚合函数

- **注意：聚合函数只会计算非NULL值,若想计算所有值，在参数中输入 '\*'**

- MAX(col_name)---计算某列的最大值
- MIN(col_name)---计算某列的最小值
- AVG(col_name)---计算某列的平均值
- SUM(col_name)---计算某列的总和
- COUNT(col_name)---计算某列有多少行





---




# 连接---在多张表检索数据

- 内连接---
```SQL
-- 语法
SELECT col_nams
FROM from_name from_alias  -- from_name_alias 是 from_name 的别名
INNER JOIN join_from_name join_from_alias
    ON conditions;
-- 别名可用可不用

-- 示例orders数据
mysql> SELECT * FROM orders;
+----+-------------+-----------+
| id | phone       | addrss    |
+----+-------------+-----------+
|  1 | 13145625478 | beijing   |
|  2 | 13894251364 | beijing   |
|  3 | 13945781654 | guangzhou |
|  4 | 18916457523 | wuhan     |
|  5 | 14615452314 | changsha  |
|  6 | 13756487529 | nanning   |
+----+-------------+-----------+
-- 示例customer的数据
mysql> SELECT * FROM customer;
+----+-----+--------+-------+
| id | age | name   | sex   |
+----+-----+--------+-------+
|  0 |  19 | yjl    | male  |
|  1 |  18 | mzh    | fmale |
|  2 |  20 | keria  | male  |
|  3 |  16 | mery   | fmale |
|  4 |  18 | mary   | fmale |
|  5 |  18 | carh   | fmale |
|  6 |  21 | Bob    | male  |
|  7 |  17 | kery   | fmale |
|  8 |  21 | Kob    | male  |
|  9 |  19 | Gropry | male  |
| 10 |  21 | Vae    | fmale |
+----+-----+--------+-------+
-- 示例连接后的数据
mysql> SELECT *  FROM orders INNER JOIN customer ON orders.id = customer.id;
+----+-------------+-----------+----+-----+-------+-------+
| id | phone       | addrss    | id | age | name  | sex   |
+----+-------------+-----------+----+-----+-------+-------+
|  1 | 13145625478 | beijing   |  1 |  18 | mzh   | fmale |
|  2 | 13894251364 | beijing   |  2 |  20 | keria | male  |
|  3 | 13945781654 | guangzhou |  3 |  16 | mery  | fmale |
|  4 | 18916457523 | wuhan     |  4 |  18 | mary  | fmale |
|  5 | 14615452314 | changsha  |  5 |  18 | carh  | fmale |
|  6 | 13756487529 | nanning   |  6 |  21 | Bob   | male  |
+----+-------------+-----------+----+-----+-------+-------+

-- 注意：连接后只显示id同的数据
```

- 跨数据库连接
```SQL
-- 语法
SELECT col_nams
FROM databse_name_from_name from_alias  -- from_name_alias 是 from_name 的别名
INNER JOIN databse_name.join_from_name join_from_alias
    ON coditions;
-- 与内连接的区别就是表的前面加了一个数据库前缀

-- 示例 显示所有数据库
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
-- 使用test数据库和testSql中的表进行连接
mysql> SELECT *  FROM testSql.customer c INNER JOIN test.orders o ON c.id = o.id;
+----+-----+-------+-------+----+-------------+-----------+
| id | age | name  | sex   | id | phone       | address   |
+----+-----+-------+-------+----+-------------+-----------+
|  1 |  18 | mzh   | fmale |  1 | 13145625478 | beijing   |
|  2 |  20 | keria | male  |  2 | 13894251364 | beijing   |
|  3 |  16 | mery  | fmale |  3 | 13945781654 | guangzhou |
|  4 |  18 | mary  | fmale |  4 | 18916457523 | wuhan     |
|  5 |  18 | carh  | fmale |  5 | 14615452314 | changsha  |
|  6 |  21 | Bob   | male  |  6 | 13756487529 | nanning   |
+----+-----+-------+-------+----+-------------+-----------+
```

 - 自连接
 ```SQL
-- 语法
SELECT col_names
FROM from_name alias    -- 两个from_name同，别名不同
INNER JOIN from_name another_alias
ON alias.key = another_alias.key; 

-- 示例 显示family表的数据
mysql> SELECT * FROM family;
+----+------+-------+-----------+-----------+
| id | name | sex   | father_id | mother_id |
+----+------+-------+-----------+-----------+
|  1 | Bob  | male  |         0 |         0 |
|  2 | mery | fmale |         0 |         0 |
|  3 | hury | male  |         1 |         2 |
|  4 | pou  | fmale |         1 |         2 |
|  5 | grye | fmale |         3 |         4 |
|  6 | nog  | male  |         3 |         4 |
+----+------+-------+-----------+-----------
-- 示例 将同一个表内的不同列的数据相关联，此次关联为id与father_id数据的关联
--      可以在同一张表中找出相关联的数据
mysql> SELECT child.name,father.name as father 
    -> FROM family child 
    -> INNER JOIN family father 
    -> ON child.father_id = father.id;
+------+--------+
| name | father |
+------+--------+
| hury | Bob    |
| pou  | Bob    |
| grye | hury   |
| nog  | hury   |
+------+--------+
 ```

 - 多表内连接
```SQL
-- 语法
SELECT col_names
FROM from_name from_alias
INNER JOIN join_from_name_1 join_from_1_alais
    ON conditions
...
INNER JOIN join_from_name_n join_from_n_alais
    ON conditions;
-- 解释：其实就是使用多个 INNER JOIN 使多个表关联起来

--示例 1.将customer表的id与orders表的id关联
--     2.将customer表的status_id与order_status的status_id关联
mysql> SELECT c.name,o.phone,o.addrss,os.name AS status
    -> FROM customer c
    -> INNER JOIN orders o
    ->     ON c.id=o.id
    -> INNER JOIN order_status os
    ->     ON c.status_id=os.status_id;
+-------+-------------+-----------+-----------+
| name  | phone       | addrss    | status    |
+-------+-------------+-----------+-----------+
| mzh   | 13145625478 | beijing   | processed |
| keria | 13894251364 | beijing   | shipped   |
| mery  | 13945781654 | guangzhou | processed |
| mary  | 18916457523 | wuhan     | shipped   |
| carh  | 14615452314 | changsha  | processed |
| Bob   | 13756487529 | nanning   | processed |
+-------+-------------+-----------+-----------+
```

- 复合连接
```SQL
-- 语法
SELECT col_names
FROM from_name alias    -- 两个from_name同，别名不同
INNER JOIN from_name another_alias
    ON conditions;  -- 复合条件，也就是多种条件的意思

-- 示例 找到孩子的父亲或者母亲
mysql> SELECT child.name,parent.name AS parents
    -> FROM family child
    -> INNER JOIN family parent
    ->     ON child.father_id=parent.id OR child.mother_id=parent.id;
+------+---------+
| name | parents |
+------+---------+
| hury | Bob     |  -- 父亲
| hury | mery    |  -- 母亲
| pou  | Bob     |      
| pou  | mery    |
| grye | hury    |
| grye | pou     |
| nog  | hury    |
| nog  | pou     |
+------+---------+
```

- 隐式连接
```SQL
-- 不过一般不建议使用这种连接
-- 语法
SELECT col_names
FROM form_1 alias_1,...,form_n alias_n  -- 直接将表都写在FROM语句后
WHERE conditions;                       -- 关联条件

-- 示例
mysql> SELECT c.name,os.name AS status
    -> FROM customer c,order_status os
    -> WHERE c.status_id=os.status_id;
+--------+-----------+
| name   | status    |
+--------+-----------+
| yjl    | shipped   |
| mzh    | processed |
| keria  | shipped   |
| mery   | processed |
| mary   | shipped   |
| carh   | processed |
| Bob    | processed |
| kery   | delivered |
| Kob    | delivered |
| Gropry | delivered |
| Vae    | shipped   |
+--------+-----------+
```

- 外连接
```SQL
-- 外连接分为两类：
-- 1. 左连接 LEFT OUTER JOIN 无论 ON 之后的条件是否成立都返回from_name的全部记录
-- 2. 右连接 RIGHT OUTER JOIN 无论ON 之后的条件是否成立都返回join_from_name的全部记录
-- 3. 尽量避免使用 右连接

-- 语法
SELECT col_names
FROM from_name from_alias,
LEFT OR RIGHT OUTER JOIN join_from_name join_from_alias
    ON conditions;

-- 示例 显示customer表，表中status_id段含有null值
mysql> SELECT * FROM customer;
+----+-----+--------+-------+-----------+
| id | age | name   | sex   | status_id |
+----+-----+--------+-------+-----------+
|  0 |  19 | yjl    | male  |         2 |
|  1 |  18 | mzh    | fmale |         1 |
|  2 |  20 | keria  | male  |      NULL |
|  3 |  16 | mery   | fmale |         1 |
|  4 |  18 | mary   | fmale |      NULL |
|  5 |  18 | carh   | fmale |         1 |
|  6 |  21 | Bob    | male  |         1 |
|  7 |  17 | kery   | fmale |      NULL |
|  8 |  21 | Kob    | male  |         3 |
|  9 |  19 | Gropry | male  |         3 |
| 10 |  21 | Vae    | fmale |         2 |
+----+-----+--------+-------+-----------+
-- 示例 显示order_status表，表中含有4个记录
mysql> SELECT * FROM order_status;
+-----------+-----------+
| status_id | name      |
+-----------+-----------+
|         1 | processed |
|         2 | shipped   |
|         3 | delivered |
|         4 | finished  |
+-----------+-----------+
-- 使用内连接使customer.status_id与order_status.status_id相关联
mysql> SELECT c.id,c.name,os.name AS status
    -> FROM customer c
    -> INNER JOIN order_status os
    ->     ON c.status_id=os.status_id;
+----+--------+-----------+
| id | name   | status    |
+----+--------+-----------+
|  0 | yjl    | shipped   |
|  1 | mzh    | processed |
|  3 | mery   | processed |
|  5 | carh   | processed |
|  6 | Bob    | processed |
|  8 | Kob    | delivered |
|  9 | Gropry | delivered |
| 10 | Vae    | shipped   |
+----+--------+-----------+
-- 发现有些顾客status_id段为null值，但是并没有显示，此时我们若想显示所有顾客可以使用左外连接
-- 示例 左外连接 customer表所有记录显示
mysql> SELECT c.id,c.name,os.name AS status
    -> FROM customer c
    -> LEFT OUTER JOIN order_status os
    ->     ON c.status_id=os.status_id;
+----+--------+-----------+
| id | name   | status    |
+----+--------+-----------+
|  1 | mzh    | processed |
|  3 | mery   | processed |
|  5 | carh   | processed |
|  6 | Bob    | processed |
|  0 | yjl    | shipped   |
| 10 | Vae    | shipped   |
|  8 | Kob    | delivered |
|  9 | Gropry | delivered |
|  2 | keria  | NULL      |
|  4 | mary   | NULL      |
|  7 | kery   | NULL      |
+----+--------+-----------+
-- 示例 右外连接 order_status表记录所有显示
mysql> SELECT c.id,c.name,os.name AS status
    -> FROM customer c
    -> RIGHT OUTER JOIN order_status os
    ->     ON c.status_id=os.status_id;
+------+--------+-----------+
| id   | name   | status    |
+------+--------+-----------+
|    0 | yjl    | shipped   |
|    1 | mzh    | processed |
|    3 | mery   | processed |
|    5 | carh   | processed |
|    6 | Bob    | processed |
|    8 | Kob    | delivered |
|    9 | Gropry | delivered |
|   10 | Vae    | shipped   |
| NULL | NULL   | finished  |
+------+--------+-----------+
-- 外连接对查询所有记录得关联非常有用
```

- 多表外连接
```SQL
-- 实质就是多张表的外连接
-- 语法
SELECT col_names
FROM from_name from_alias,
LEFT OR RIGHT OUTER JOIN join_from_name_1 join_from_alias_1
    ON conditions;
...
LEFT OR RIGHT OUTER JOIN join_from_name_n join_from_alias_n
    ON conditions;    
-- 可以自己举例
```

- 自外连接
```SQL
-- 语法
SELECT col_names
FROM from_name alias    -- 两个from_name同，别名不同
LEFT OR RIGHT OUTER JOIN from_name another_alias
    ON conditions;

-- 示例 family表数据全部显示
mysql> SELECT child.name,mother.name AS mother
    -> FROM family child
    -> LEFT OUTER JOIN family mother
    ->     ON child.mother_id=mother.id;
+------+--------+
| name | mother |
+------+--------+
| Bob  | NULL   |
| mery | NULL   |
| hury | mery   |
| pou  | mery   |
| grye | pou    |
| nog  | pou    |
+------+--------+
```

- USING子句
```SQL
-- 语法
USING (col_name) 相当于 ON from.col_name=join_from.col_name
-- 也就是说在from表与join_from表中存在相同的col_name列
-- 那就可以使用USING (col_name)来替代ON from.col_name=join_from.col_name

SELECT col_names
FROM from_name from_alias
JOIN join_from_name join_from_alias     -- 内连接 或者 外连接
    USING (col_name_1,...,col_name_n);    -- 可以使用多个相同列

-- 示例 customer表和order_status表都含有列status_id
mysql> SELECT c.id,c.name,os.name AS status
    -> FROM customer c
    -> LEFT OUTER JOIN order_status os
    ->     USING (status_id);
+----+--------+-----------+
| id | name   | status    |
+----+--------+-----------+
|  1 | mzh    | processed |
|  3 | mery   | processed |
|  5 | carh   | processed |
|  6 | Bob    | processed |
|  0 | yjl    | shipped   |
| 10 | Vae    | shipped   |
|  8 | Kob    | delivered |
|  9 | Gropry | delivered |
|  2 | keria  | NULL      |
|  4 | mary   | NULL      |
|  7 | kery   | NULL      |
+----+--------+-----------+
```

- 自然连接---**NATURAL**
```SQL
-- 关键字 NATURAL
-- 使用NATURAL修饰JOIN，使得数据库搜索引擎自动寻找相同列作为连接条件
-- 故使用NATURAL返回得数据可能并不是你实际想要的结果

-- 语法
SELECT col_names
FROM from_name from_alias
NATURAL JOIN join_from_name join_from_alias;
-- 不推荐使用这个关键字
```

- 交叉连接---**CROSS**
```SQL
-- 关键字 CROSS
-- 交叉连接使相连接的表进行交叉组合
-- 假设 from_1有2条记录，from_2有3条记录
-- 交叉的意思就是让from_1中的每条记录去组合from_2中的每条记录
-- 即form_1中的一条记录可以和from_2中的3条记录进行组合
-- 则最后返回的数据为 2 x 3 = 6 条记录

-- 语法1 显式语法
SELECT col_names
FROM from_name from_alias
CROSS JOIN join_from_name join_from_alias;

-- 语法2 隐式语法(不推荐这样写)
SELECT col_names
FROM from_name from_alias, join_from_name join_from_alias;

-- 示例
mysql> SELECT c.name ,os.name AS status
    -> FROM customer c
    -> CROSS JOIN order_status os;
+--------+-----------+
| name   | status    |
+--------+-----------+
| yjl    | processed |
| yjl    | shipped   |
| yjl    | delivered |
| yjl    | finished  |
| mzh    | processed |
| mzh    | shipped   |
         .
         .
         .
| Gropry | finished  |
| Vae    | processed |
| Vae    | shipped   |
| Vae    | delivered |
| Vae    | finished  |
+--------+-----------+
```

- 联合---**UNION**
```SQL
-- 合并查询结果，实质就是两张表返回结果的尾首相接
-- 返回结果的列名以第一个表查询的列名为准
-- 语法
SELECT col_names    -- 注意：查询列的数量要相等
FROM from_name_1 alias_1
UNION
SELECT col_names
FROM from_name_2 alias_2;
-- 没啥用的关键字
```
