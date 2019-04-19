---
title: Zookeeper集群搭建
date: 2019-04-19 14:04:00
tags:
    - Zookeeper
categories:
    - 安装教程
---
{% blockquote %}
 为了用到Solr，记录一下Zookeeper 的集群配置。
{% endblockquote %}
### 下载地址
    官网首页： https://zookeeper.apache.org/
    下载地址： http://mirror.bit.edu.cn/apache/zookeeper/
#### 集群规划
在hadoop5、hadoop6和hadoop7三个节点上部署Zookeeper，三个节点都已安装jdk。

|IP	|节点名称	|Zookeeper	|
| ------------- |:-------------------:|:------------------:|
|192.168.100.21 |hadoop5|		zookeeper-3.4.14|
|192.168.100.22	|hadoop6 |zookeeper-3.4.14|
|192.168.100.23	|hadoop7|zookeeper-3.4.14|
#### 解压安装
    （1）解压zookeeper安装包到/opt/module/目录下
    
    [admin@node21 software]$ tar -zxvf zookeeper-3.4.14.tar.gz -C /opt/module/
    （2）在/opt/module/zookeeper-3.4.14/这个目录下创建Data
    
    [admin@node21 zookeeper-3.4.14]# mkdir  Data
    （3）重命名/opt/module/zookeeper-3.4.14/conf这个目录下的zoo_sample.cfg为zoo.cfg
    
    [admin@node21 conf]# mv zoo_sample.cfg zoo.cfg
#### 配置zoo.cfg文件
    具体配置，修改dateDir,添加日志存放目录
    
    dataDir=/opt/module/zookeeper-3.4.14/Data
    dataLogDir=/opt/module/zookeeper-3.4.14/logs
    末尾增加如下配置
    
    server.1=node21:2888:3888
    server.2=node22:2888:3888
    server.3=node23:2888:3888
#### 集群配置
    （1）在/opt/module/zookeeper-3.4.14/Data目录下创建一个myid的文件
    
    [admin@node21 Data]# sudo touch myid
    （2）编辑myid文件， 在文件中添加与server对应的编号：如 1   
    
    [admin@node21 Data]# vi myid
    （3）拷贝配置好的zookeeper到其他机器上
    
    [admin@node21 module]# scp -r zookeeper-3.4.14/ admin@node22:/opt/module/
    [admin@node21 module]# scp -r zookeeper-3.4.14/ admin@node23:/opt/module/
    并分别修改node22，node23中myid文件中内容为2、3
    
    [admin@node22 Data]# echo 2 > myid
    [admin@node23 Data]# echo 3 > myid
#### 启动集群
    （1）分别启动zookeeper
    
    [admin@node21 zookeeper-3.4.14]# bin/zkServer.sh start
    [admin@node22 zookeeper-3.4.14]# bin/zkServer.sh start
    [admin@node23 zookeeper-3.4.14]# bin/zkServer.sh start
    （2）查看状态
    
    复制代码
    [admin@node21 zookeeper-3.4.14]# bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /opt/module/zookeeper-3.4.14/bin/../conf/zoo.cfg
    Mode: follower
    [admin@node22 zookeeper-3.4.14]# bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /opt/module/zookeeper-3.4.14/bin/../conf/zoo.cfg
    Mode: follower
    [admin@node23 zookeeper-3.4.14]# bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /opt/module/zookeeper-3.4.14/bin/../conf/zoo.cfg
    Mode: leader
    复制代码
    （3）停止zookeeper
    
    [admin@node21 zookeeper-3.4.14]$ bin/zkServer.sh stop
#### 配置环境变量
[admin@node21 zookeeper-3.4.14]$ sudo vi /etc/profile 
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.14 
export PATH=$PATH:$ZOOKEEPER_HOME/bin
#### zoo.cfg配置参数解读
Server.A=B:C:D。

A是一个数字，表示这个是第几号服务器；

B是这个服务器的ip地址；

C是这个服务器与集群中的Leader服务器交换信息的端口；

D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。

1）tickTime=2000：通信心跳数

tickTime：通信心跳数，Zookeeper服务器心跳时间，单位毫秒

Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。

它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)

2）initLimit=10：LF初始通信时限

集群中的follower跟随者服务器(F)与leader领导者服务器(L)之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。

投票选举新leader的初始化时间

Follower在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态。

Leader允许F在initLimit时间内完成这个工作。

3）syncLimit=5：LF同步通信时限

集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，

Leader认为Follwer死掉，从服务器列表中删除Follwer。

在运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。

如果L发出心跳包在syncLimit之后，还没有从F那收到响应，那么就认为这个F已经不在线了。

4）dataDir：数据文件目录+数据持久化路径

保存内存数据库快照信息的位置，如果没有其他说明，更新的事务日志也保存到数据库。

 5）clientPort=2181：客户端连接端口

监听客户端连接的端口