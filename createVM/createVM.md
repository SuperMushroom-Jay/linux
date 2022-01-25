# 在虚拟机上安装centOS系统
---
## 一、下载centOS镜像
[centOS镜像网址]:https://www.centos.org/ "镜像网址"

1. 点击[centOS镜像网址]下载对应的centOS镜像
2. 点击红色→指向的区域 ![step-1](res/vm_step_1.png)
3. 此次下载centOS-7版本 ![step-2](res/vm_step_2.png)
4. 点击第一个链接 ![step-3](res/vm_step_3.png)
5. 点击下载![step-4](res/vm_step_4.png)
---
## 二、新建VM虚拟机
1. 打开VM，然后点击 “文件->新建虚拟机” 或者点击主页上的 “创建新的虚拟机” ![step-5](res/vm_step_5.png)
2. 在选项卡中选择"自定义(高级)" ![step-6](res/vm_step_6.png)
3. 选择默认兼容性，点击下一步 ![step-7](res/vm_step_7.png)
4. 选择稍后安装操作系统 ![step-8](res/vm_step_8.png)
5. 操作系统选择linux![step-9](res/vm_step_9.png)
6. 版本选择你之前下载的镜像版本![step-10](res/vm_step_10.png)
7. 命名虚拟机以及选择虚拟机的安装位置 ![step-11](res/vm_step_11.png)
8. 为虚拟机配置处理器，根据自己电脑的配置选择 ![step-12](res/vm_step_12.png)
9. 其中处理器内核总数不能超过自己主机的逻辑处理器总数![step-13](res/vm_step_13.png)
10. 为虚拟机配置运行内存，不能超过自己主机的最大运行内存 ![step-14](res/vm_step_14.png)
12. 为虚拟机配置网络类型，默认选项 ![step-15](res/vm_step_15.png)
13. 默认选项 ![step-16](res/vm_step_16.png)
14. 默认选项 ![step-17](res/vm_step_17.png)
15. 默认选项 ![step-18](res/vm_step_18.png)
16. 为虚拟机配置磁盘大小 ![step-19](res/vm_step_19.png)
17. 选择虚拟机磁盘安装位置 ![step-20](res/vm_step_20.png)、
18. 点击完成即可 ![step-21](res/vm_step_21.png)

## 三、在VM中安装linux
1. 由于之前选择了稍后安装操作系统，所以现在需要选择相应的镜像文件
2. 点击编辑虚拟机设置![step-22](res/vm_step_22.png)
3. 然后点击使用ISO镜像文件 ![step-23](res/vm_step_23.png)
4. 浏览选择相应的iso镜像文件即可![step-24](res/vm_step_24.png)

## 四、linux安装步骤
1. 键盘选择第一个install centOS 7 ![centOS-1](res/centOS_1.png)
2. 回车 ![centOS-21](res/centOS_2.png)
3. 选择语言![centOS-3](res/centOS_3.png)
4. 基础设置，选择软件选择安装桌面版本 ![centOS-4](res/centOS_4.png)
5. 设置root以及用户 ![centOS-5](res/centOS_5.png)
6. 重启虚拟机 ![centOS-6](res/centOS_6.png)