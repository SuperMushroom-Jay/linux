# 配置nginx网络代理服务
---
-  **注意：全部命令应该在root用户权限下执行。**
```
//登录root用户，然后输入密码（密码不可见）
[root@ecs-7 ~]# su root
Password:
```
- 然后使用指令yum指令下载nginx
```
//下载安装nginx
[root@ecs-7 ~]# yum install nginx

//查看nginx的版本
[root@ecs-7 ~]# nginx -v
nginx version: nginx/1.20.1
```
- 查看服务器端口占用情况
```
[root@ecs-7 ~]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      7578/nginx: master  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      9357/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      965/master                
tcp6       0      0 :::80                   :::*                    LISTEN      7578/nginx: master  
tcp6       0      0 :::22                   :::*                    LISTEN      9357/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      965/master          
//可以看到nginx占用了80端口
```
- 配置防火墙
```
//查看防火墙状态
[root@ecs-7 ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: active (running) since Wed 2022-03-09 11:06:42 CST; 11h ago
    ...
    ...
//显示 running 代表防火墙正在运行

//查寻防火墙80端口的开启情况
[root@ecs-7 ~]# firewall-cmd --query-port=80/tcp
no
//显示 no 代表没用开启80端口

//永久开启80端口
[root@ecs-7 ~]# firewall-cmd --permanent --add-port=80/tcp
success
//显示 success 代表开启80端口成功

//重启防火墙
[root@ecs-7 ~]#firewall-cmd --reload

//再次查询80端口开启情况
[root@ecs-7 ~]# firewall-cmd --query-port=80/tcp
yes
//显示yes代表成功
```
- 配置服务器安全组，找到自己购买服务器的控制台，添加安全组，开启80端口
- 完成以上工作后，即可在浏览器上使用 http://ipaddress 来访问nginx默认网页
- 浏览器默认显示的html文件在路径： /usr/share/nginx/html/ 若只想显示静态网页，在此放置你的网页文件即可
- 假如你想在其他文件夹存在你的网页文件，现需要修改nginx的配置文件，其配置文件为：/etc/nginx/nginx.conf
```
[root@ecs-7 ~]# cat /etc/nginx/nginx.conf
...
server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;
...
//修改 /usr/share/nginx/html 为你想存在网页文件的路径
```