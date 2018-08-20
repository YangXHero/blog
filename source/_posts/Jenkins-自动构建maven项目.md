---
title: Jenkins 自动构建maven项目
date: 2018-05-04 14:00:10
tags:
    - CentOS 服务搭建
    - Jenkins 使用
categories:
    - CentOS
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 
height=86 src="//music.163.com/outchain/player?type=2&id=247415&auto=1&height=66"></iframe>

{% blockquote %}
Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。
{% endblockquote %}

#### 1. Jenkins安装构建Maven项目时必要的插件。
> jenkins 》 系统管理 》 插件管理 》 可选插件 》 输入搜索安装

`publish over SSH` `Maven Integration plugin`
#### 2. 配置项目构建需要的工具。
> jenkins 》 系统管理 》 全局工具管理

配置jdk   maven   git 路径就可以。

{% asset_img 2.png 工具地址 %}
{% asset_img 3.png 工具地址 %}
#### 3. 配置ssh server信息。用于构建完成执行远程命令。
> jenkins 》 系统管理 》 系统设置 》 应该是最底下。直接拉到最底下，如下图配置自己的信息。

{% asset_img 6.png 配置ssh %}
#### 4. 创建一个新任务。
{% asset_img 1.png 创建任务 %}

#### 5. 配置源码位置。
{% asset_img 4.png 源码位置 %}
#### 6. 构建命令设置。
{% asset_img 5.png 构建命令 %}

#### 7. 配置完成后操作。
> ssh 免密登录: {% post_link SSH-免密钥登录 %} 。

    1.先将自己的jar或者war传至远程目录。
    2.ssh执行command 或者 在远程服务器新建 shell命令文件。直接执行。
{% asset_img 7.png 配置 %}

#### 8. 扩展——基于webhook实现自动化构建。
> 1.jenkins 搜索并安装插件 : Gogs

> 2.jenkins 任务管理，配置密匙。

{% asset_img 8.png 配置 %}    
> 3.gogs配置web钩子。 

{% asset_img 9.png 配置 %} 