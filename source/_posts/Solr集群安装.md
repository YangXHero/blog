---
title: Solr集群安装
date: 2019-04-19 15:14:51
tags:
    - Solr
categories:
    - 安装教程
---
#### 安装环境，集群部署
|IP	|节点名称	|Zookeeper	|Solr|
| ------------- |:-------------------:|:------------------:|:------------------:|
|192.168.100.21 |hadoop5|		zookeeper-3.4.14|solr-7.7.1.tgz|
|192.168.100.22	|hadoop6 |zookeeper-3.4.14|solr-7.7.1.tgz|
|192.168.100.23	|hadoop7|zookeeper-3.4.14|solr-7.7.1.tgz|
#### 解压安装包，运行安装脚本

1. tar -zxvf solr-7.7.1.tgz
2. tar zxf solr-7.7.1.tgz solr-7.7.1/bin/install_solr_service.sh --strip-components=2

        上一个命令将install_solr_service.sh脚本从存档中提取到当前目录中
        
        mkdir /opt/module/solr2 
        mkdir /opt/module/solr

3. 运行服务安装脚本:
    
        bash ./install_solr_service.sh solr-7.7.1.tgz -i /opt/module/solr -d /opt/module/solr/solrhome -u solr -s solr -p 8983
        bash ./install_solr_service.sh solr-7.7.1.tgz -i /opt/module/solr2 -d /opt/module/solr2/solrhome -u solr -s solr2 -p 8984
        
4. 修改对应jetty服务的端口8983/8984
   
        vi  /opt/module/solr/solrhome/data/solr.xml 
        vi  /opt/module/solr2/solrhome/data/solr.xml 
        vi  /opt/module/solr/solrhome/data/solr.xml 
        vi  /opt/module/solr2/solrhome/data/solr.xml 
#### 配置zk启动优先级
1. zookeeper自动启动 : vi /etc/init.d/zookeeper
        
        #!/bin/bash
        #chkconfig:2345 20 90
        #description:zookeeper
        #processname:zookeeper
        export JAVA_HOME=/opt/module/jdk1.8
        export ZOO_LOG_DIR=/opt/module/zookeeper-3.4.14/logs
        case $1 in
                start) su root /opt/module/zookeeper-3.4.14/bin/zkServer.sh start;;
                stop) su root /opt/module/zookeeper-3.4.14/bin/zkServer.sh stop;;
                status) su root /opt/module/zookeeper-3.4.14/bin/zkServer.sh status;;
                restart) su root /opt/module/zookeeper-3.4.14/bin/zkServer.sh restart;;
                *) echo "require start|stop|status|restart" ;;
        esac
     
2. chmod +x /etc/init.d/zookeeper
3. chkconfig --add zookeeper

#### 关联solr集群与zk集群
1. vi /etc/default/solr.in.sh
2. vi /etc/default/solr2.in.sh

        修改如下信息（对应主机host注意更改）：
        ZK_HOST="node21:2181,node22:2181,node23:2181/solr"
        SOLR_HOST="192.168.100.21"

3. 首次连接需要创建节点管理目录
        
        ./solr/bin/solr zk mkroot /solr -z hadoop5:2181,hadoop6:2181,hadoop7:2181
4. 使用Solr的ZooKeeper CLI上传solr配置信息到zk节点 
        
        sh solr/solr-7.7.1/server/scripts/cloud-scripts/zkcli.sh -zkhost hadoop5:2181,hadoop6:2181,hadoop7:2181 -cmd upconfig -confdir /opt/module/solr/solr-7.7.1/server/solr/configsets/_default/conf -confname myconf
5. 重新启动
        
        service solr restart
        service solr2 restart
#### 配置solr集群的分片规则

    ./solr/bin/solr create -c collection1 -n collection1 -shards 2 -replicationFactor 2 -p 8983 -force
    ./solr2/bin/solr create -c collection2 -n collection2 -shards 2 -replicationFactor 2 -p 8984 -force
    
    删除操作
      ./solr/solr/bin/solr delete -c collection2 
    
    参数说明：
    
    -c <name> 要创建的核心或集合的名称（必需）。
    
    -n <configName> 配置名称，默认与核心或集合的名称相同。
    
    -p <port> 发送create命令的本地Solr实例的端口; 默认情况下，脚本会尝试通过查找正在运行的Solr实例来检测端口。
    
    -s <shards> 要么 -shards 将集合拆分为的分片数，默认为1; 仅适用于Solr在SolrCloud模式下运行的情况。
    
    -rf <replicas> 要么 -replicationFactor 集合中每个文档的副本数。默认值为1（无复制）。
    
    -force 如果尝试以“root”用户身份运行create，则脚本将退出并显示运行Solr或针对Solr的操作作为“root”的警告可能会导致问题。可以使用-force参数覆盖此警告。
    
    -d <confdir> 配置目录。默认为_default。
   
#### ik中文分词器
1. jar包复制到各个节点

        cp ikanalyzer-solr5/ik-analyzer-solr5-5.x.jar ikanalyzer-solr5/solr-analyzer-ik-5.1.0.jar /opt/module/solr/solr-7.7.1/server/solr-webapp/webapp/WEB-INF/lib/
2. 各个节点执行

        mkdir -p /opt/module/solr/solr-7.7.1/server/solr-webapp/webapp/WEB-INF/classes
        cp ext.dic IKAnalyzer.cfg.xml stopword.dic /opt/module/solr2/solr-7.7.1/server/solr-webapp/webapp/WEB-INF/classes/
        
3. 更改配置文件

        vim /opt/module/solr/solr-7.7.1/server/solr/configsets/_default/conf/managed-schema
        <!--IK中文分词器-->
         <fieldType name="text_ik" class="solr.TextField">  
                <analyzer type="index" useSmart="false"
                    class="org.wltea.analyzer.lucene.IKAnalyzer" />
                <analyzer type="query" useSmart="true"
                    class="org.wltea.analyzer.lucene.IKAnalyzer" />
        </fieldType>
4. 更新配置文件

        sh /opt/module/solr/solr-7.7.1/server/scripts/cloud-scripts/zkcli.sh -zkhost hadoop5:2181,hadoop6:2181,hadoop7:2181 -cmd upconfig -confdir /opt/module/solr/solr-7.7.1/server/solr/configsets/_default/conf/ -confname myconf
        