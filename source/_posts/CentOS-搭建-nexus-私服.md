---
title: CentOS 搭建 nexus 私服
date: 2018-04-18 17:56:44
tags:
    - CentOS 服务搭建
categories:
    - CentOS
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 
src="//music.163.com/outchain/player?type=2&id=27646196&auto=1&height=66"></iframe>

#### 1. 安装JDK.
安装教程：{% post_link JDK安装 %}
#### 2. 推荐新建nexus用户管理nexus。
    adduser nexus
    passwd nexus
    su - nexus

#### 3. 下载nexus。本文nexus版本：2.14.8-01  [下载地址](https://www.sonatype.com/download-sonatype-trial) 
{% asset_img 01.png 下载地址 %}

或者直接下载 `wget https://sonatype-download.global.ssl.fastly.net/nexus/oss/nexus-2.14.1-01-bundle.tar.gz`

#### 4. 解压文件：`tar -zxvf nexus文件`
{% asset_img 02.png 解压 %}
第一个文件夹是核心文件，第二个文件夹用来存储下载下来的jar。
{% asset_img 03.png 文件夹 %}
#### 5. 进入nexus/nexus-2.14.8-01/conf目录下，编辑nexus.properties文件，命令：`vim nexus.properties`

{% asset_img 04.png 修改配置文件 %}
#### 6. 进入nexus/nexus-2.14.8-01/bin目录，`vim nexus`，修改启动用户。

{% asset_img 05.png 修改启动用户 %}
#### 7. 启动nexus  `./nexus start`
#### 8. 访问nexus  `地址:端口/nexus`
{% asset_img 06.png index %}

搭建完成。

#### 9. maven settings.xml文件修改。
    
```xml
<!--一、localRepository-->
```
{% asset_img 07.png localRepository %}
```xml
<!--二、mirrors下新建mirror地址-->
<mirror>
  <id>internal-repository</id>
  <name>Maven Repository Manager running on www.ctfo.com</name>
  <url>http://你的私服地址/nexus/content/groups/public/</url>
  <mirrorOf>*</mirrorOf>
</mirror>


<!--三、maven项目 deploey jar文件。（IDE）-->
<!--在maven settings.xml文件，新增server配置。-->
<server>
   <id>thirdparty</id>
   <username>admin</username>
   <password>admin123</password>
</server>
<!--在项目pom文件加-->
<distributionManagement>
    <repository>
        <id>thirdparty</id>
        <name>nexus</name>
        <url>http://你的私服地址/nexus/content/repositories/thirdparty/</url>
    </repository>
</distributionManagement>
<!--然后deploey就到了第三方库。-->
```