---
title: Redis集群搭建
date: 2018-10-26 11:48:54
tags:
    - Redis
categories:
    - 安装教程
---

#### 1.安装搭建集群所需要的环境。
    命令行运行：
         yum -y install gcc psmisc

#### 2. 下载redis 安装文件 并解压。
    wget http://download.redis.io/releases/redis-4.0.1.tar.gz
    tar -zxvf 文件名。

#### 3. 进入解压好的文件目录，安装。
    make MALLOC=libc
    make & make install
* 如果运行make报错，可能如下：
* gcc依赖：yum install gcc gcc-c++ ncurses-devel
* cd deps下。执行。
    ```shell
    make hiredis
    make jemalloc
    make lua
    make linenoise
    ```

#### 4. 搭建集群 --- 创建集群所需目录
    cd /usr/local
    mkdir redis-cluster
    cd redis-cluster
    mkdir 7001 7002 7003 7004 7005 7006

#### 5. 搭建集群 --- 修改配置文件
    cp /usr/local/redis-4.0.1/redis.conf 7001/
    vim 7001/redis.conf
    修改配置文件的选项如下
        port 7001
        daemonize yes
        bind 127.0.0.1 注释掉
        protected-mode yes  改为  no    如果yes 不允许外网访问。
        pidfile  /var/run/redis_7001.pid
        cluster-enabled yes
        cluster-config-file nodes-7001.conf
        cluster-node-timeout  15000               //请求超时  默认15秒，可自行设置
        appendonly  yes
    如果要加密码的话：
        masterauth 你的密码
        requirepass 你的密码
    修改完成一个之后，复制7001下的文件到其他五个文件夹。
        cp 7001/redis.conf 7002/
        cp 7001/redis.conf 7003/
        cp 7001/redis.conf 7004/
        cp 7001/redis.conf 7005/
        cp 7001/redis.conf 7006/
    然后全文搜索修改端口号就行。
        vim 7002/redis.conf
        :%s/7001/7002/g
        改五次。

#### 5. 搭建集群 --- 安装集群所需的依赖环境。
    yum -y install ruby rubygems
    gem install redis
    如果提示： redis requires Ruby version >= 2.2.2.
    CentOS7 yum库中ruby的版本支持到 2.0.0,
    可gem 安装redis需要最低是2.2.2,自己编译的ruby源码。
    解决方法如下：
        yum -y install curl
        curl -L get.rvm.io | bash -s stable
        find / -name rvm -print
        source /usr/local/rvm/scripts/rvm
        rvm list known
        rvm install 2.4.1
        rvm use 2.4.1
        rvm use 2.4.1 --default
        rvm remove 2.0.0
        ruby --version
    如果不成功，百度搜升级ruby。
        gem install redis

#### 6.如果加了密码，修改client.rb文件。
    find / -name client.rb
    找到路径中包含redis版本号那个。
    vim 文件。
{% asset_img 001.png 修改连接密码 %}

#### 7. 批处理文件启动redis。
    vim redis-start.sh
```shell
cd /usr/local/redis-cluster/7001/
redis-server redis.conf
cd /usr/local/redis-cluster/7002/
redis-server redis.conf
cd /usr/local/redis-cluster/7003/
redis-server redis.conf
cd /usr/local/redis-cluster/7004/
redis-server redis.conf
cd /usr/local/redis-cluster/7005/
redis-server redis.conf
cd /usr/local/redis-cluster/7006/
redis-server redis.conf
```
    chmod +x redis-start.sh
    vim redis-remove.sh
```shell
cd /usr/local/redis-cluster/7001/
rm -rf appendonly.aof
rm -rf nodes-7001.conf
cd /usr/local/redis-cluster/7002/
rm -rf appendonly.aof
rm -rf nodes-7002.conf
cd /usr/local/redis-cluster/7003/
rm -rf appendonly.aof
rm -rf nodes-7003.conf
cd /usr/local/redis-cluster/7004/
rm -rf appendonly.aof
rm -rf nodes-7004.conf
cd /usr/local/redis-cluster/7005/
rm -rf appendonly.aof
rm -rf nodes-7005.conf
cd /usr/local/redis-cluster/7006/
rm -rf appendonly.aof
rm -rf nodes-7006.conf
```
    chmod +x redis-remove.sh

#### 8. 集群搭建，启动集群
    ./redis-start.sh
    如果是云服务器，先放开端口 7001-7006  和 17001-17006
    ./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
    将127.0.0.1 改成你的ip



