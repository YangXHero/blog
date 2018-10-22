---
title: Mysql57 rpm 安装
date: 2018-10-22 11:38:13
tags:
    - CentOS 服务搭建
    - Mysql57 搭建。
categories:
    - CentOS
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86
src="//music.163.com/outchain/player?type=2&id=546279760&auto=1&height=66"></iframe>

#### 1.下载和安装mysql源
先下载 mysql源安装包
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
-bash: wget: 未找到命令
我们先安装下wget
yum -y install wget
然后执行 wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

安装本地mysql源
yum -y localinstall mysql57-community-release-el7-11.noarch.rpm

#### 2.在线安装Mysql
yum -y install mysql-community-server
下载的东西比较多 要稍微等会；

#### 3.启动Mysql服务
systemctl start mysqld

#### 4.设置开机启动
systemctl enable mysqld
systemctl daemon-reload

#### 5.修改root本地登录密码
mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个临时的默认密码。
vi /var/log/mysqld.log
{% asset_img 001.png 找临时密码 %}
[root@localhost ~]#  mysql -u root -p
Enter password:
输入临时密码 进入mysql命令行；
#### 6.修改密码。
首先，修改validate_password_policy参数的值
mysql> set global validate_password_policy=0;
再修改密码的长度
mysql> set global validate_password_length=1;
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '******';  自定义密码。
Query OK, 0 rows affected (0	.00 sec)
#### 7.设置允许远程登录
Mysql默认不允许远程登录，我们需要设置下，并且防火墙开放3306端口；
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Pingan369@' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.01 sec)
mysql> exit;

#### 8.Mysql57 需要修改sql_mode。
        vi /etc/my.cnf
        在[mysqld]下面添加如下列：
        sql_mode=STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

#### 9.重新启动MySQL。
systemctl restart mysqld
