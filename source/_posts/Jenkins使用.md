---
title: Jenkins使用
date: 2018-04-18 17:56:44
tags:
    - CentOS 服务搭建
    - Jenkins 使用
categories:
    - CentOS
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" 
width=330 height=86 src="//music.163.com/outchain/player?type=2&id=25638632&auto=1&height=66"></iframe>
大学时候偶然听到这首歌，视频不错。

#### 1. 新建用户
```text
useradd jenkins
passwd jenkins
```
#### 2. 安装jdk1.8
安装教程：{% post_link JDK安装 %}
#### 3. 用jenkins用户安装tomcat8
```text
wget http://124.205.69.162/files/A0460000056AFFBC/mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.14/bin/apache-tomcat-8.5.14.tar.gz
tar -zxvf apache-tomcat-8.5.14.tar.gz
```
#### 4. 安装git和maven.（不要安装maven3.5版本，有冲突），我用的root用户
```text
一、上传maven包，并解压。（tar -zxvf）
二、修改maven的setting配置文件。
三、将maven安装路径放入环境变量。
四、yum install git。安装git
```
#### 5. 下载Jenkins并上传至tomcat的webapps目录下
 将Jenkins.war上传至Jenkins的tomcat下。然后启动。
 [Jenkins下载](https://jenkins.io/download/)

#### 6. 开放tomcat端口。启动完成后地址栏访问Jenkins.
{% asset_img 1.png 启动完成 %}
出现上图，表示启动成功。
#### 7. 根据提示，在用户下找这个文件，将文件中字符串复制入输入框即可。
{% asset_img 2.png 输入密匙 %}
#### 8. 安装插件 。新手选择推荐安装即可。反之随意。
{% asset_img 3.png 安装插件 %}
#### 9. 等待安装完成。新建用户。
{% asset_img 4.png 新建一个用户 %}
然后就可以根据自己的需求下载插件使用了。