---
title: SSH 免密钥登录
date: 2018-05-04 17:35:59
tags:
    - CentOS 服务搭建
    - SSH
categories:
    - CentOS
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86
 src="//music.163.com/outchain/player?type=2&id=247416&auto=1&height=66"></iframe>
 
#### SSH免密钥登录
 > A为本地主机(即用于控制其他主机的机器,jenkins服务器) ;
 
 > B为远程主机(即被控制的机器Server，jenkins运行之后发布项目的服务器), 假如ip为192.168.1.100 ;

 > A和B的系统都是Linux

```text
在A上的命令:
1、 ssh-keygen -t rsa (连续三次回车,即在本地生成了公钥和私钥,不设置密码)
2、 ssh root@192.168.1.100 "mkdir .ssh;chmod 0700 .ssh" (需要输入密码， 注:必须将.ssh的权限设为700)，如果提示.ssh已经存在，直接ssh root@192.168.1.100  "chmod 0700 .ssh"
3、scp ~/.ssh/id_rsa.pub root@192.168.1.100:.ssh/id_rsa.pub (需要输入密码)
```
```text
在B上的命令:
4、 touch /root/.ssh/authorized_keys (如果已经存在这个文件, 跳过这条)
5、 chmod 600 ~/.ssh/authorized_keys  (# 注意： 必须将~/.ssh/authorized_keys的权限改为600, 该文件用于保存ssh客户端生成的公钥，可以修改服务器的ssh服务端配置文件/etc/ssh/sshd_config来指定其他文件名）
6、cat /root/.ssh/id_rsa.pub  >> /root/.ssh/authorized_keys (将id_rsa.pub的内容追加到 authorized_keys 中, 注意不要用，否则会清空原有的内容，使其他人无法使用原有的密钥登录)
7、回到A机器:  ssh root@192.168.1.100 (不需要密码, 登录成功)
```