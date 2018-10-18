---
title: JDK安装
date: 2018-04-18 17:56:44
tags:
    - CentOS 服务搭建
categories:
    - CentOS
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 
src="//music.163.com/outchain/player?type=2&id=247421&auto=1&height=66"></iframe>

注意。rpm包必须在官网或其他地址下载。直接wget 官网下载链接会报错。

#### 1. 这里采用rpm安装方式。先下载rpm文件。

#### 2. 下载到本地，通过shell使用rz命令上传至服务器。
    如果出现如下错误：`-bash: rz: command not found`

    安装lrzsz：`yum -y install lrzsz`

    现在就可以正常使用rz、sz命令上传、下载数据了。
    使用方法：
    上传文件: rz 会弹出对话框，选择文件上传
#### 3. 上传完成之后，执行命令`rpm -ivh 您的jdk包`。等待安装完成。安装完成之后，路径在`/usr/java/`下。
#### 4. 配置环境变量。`vim /etp/profile`
    在文件最下方加入：
    JAVA_HOME=/usr/java/jdk1.8.0_151
    JRE_HOME=/usr/java/jdk1.8.0_151/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export PATH
#### 5. 验证是否配置完成。`java -version`