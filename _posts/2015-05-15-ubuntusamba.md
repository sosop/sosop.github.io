---
layout: post
title: "ubuntu安装samba"
description: ""
category: OS
tags: []
---
{% include JB/setup %}

1.安装组件

	sudo aptitude install samba smbfs smbclient  

2.修改配置文件  

	sudo vim /etc/samba/smb.conf

__全局设置部分__  
security = user  
这行设置了samba的安全等级，Samba一共可以设置四个安全登记，由底到高分别为：  
share：这个选项表示任何人都可以不需要输入密码登录。  
user： 这个是Samba的默认级别，要求每个用户必须输入密码才能登录。  
server：user级别的密码都是保存在本机上，而server级别的密码和用户名都保存在另一台主机上。  
domain：这个级别要求网络里必须有一台Windows的域控制器，验证工作由域控制器来完成。  
需要注意，只要输入用户名和密码的级别，其用户名一定首先也是Linux系统内的用户。  
  
workgroup ＝ MSHOME  
这部分是Windows主机的工作组明，Windows主机必须在同一个工作组中  
server string = %h server(Samba,Ubuntu)  
这个选项是显示在Windows上的信息，可以自定义，其中％h为Samba配置文件中的变量，代表了主机名，即使用hostname命令得到的主机名  
  
map to guest = bad user  
当 security = user 时，这个选项必须注释掉，如：# map to guest = bad user  
  
__共享设置部分__  
添加一个共享文件夹设置如下  
[share]  
共享文件名，不需要与实际文件名一致  
comment = my share directory
对这个共享分支的描述  
path = /home/share  
系统的共享目录，必须为绝对路径  
public = yes  
是否允许所有人都能够看到此目录，no为看不到  
writable = yes  
是否允许用户在此目录下可写，no为不可些，如果可写，还需要目录具有写权限  
read only = yes  
设置用户是否只读  
vaild users = username  
设置只有 username 用户有效  
create mask = 0700  
directory mask =0700  
force user =nobody  
force group = nogroup  
  
3.SAMBA 用户管理  

	sudo smbpasswd -a username

－a: 新添加一个Samba用户  
－d:禁用一个Samba用户  
－e:使禁用的Samba用户解禁  
  
4.启动 SAMBA 服务  

	# 停止 SAMBA 服务
	sudo /etc/init.d/smbd stop
	# 启动 SAMBA 服务
	sudo /etc/init.d/smbd start
	# 重新启动
	sudo /etc/init.d/smbd restart

