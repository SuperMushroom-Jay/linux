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
