---
title: Vultr 搭建 VPS 外网ssr服务器
date: 2018-11-06 11:07:24
tags:
    - CentOS 服务搭建
categories:
    - CentOS
---
<iframe 
    frameborder="no" border="0" marginwidth="0" marginheight="0" width=330
 height=86 src="//music.163.com/outchain/player?type=2&id=25714146&auto=1&height=66"></iframe>

{% blockquote %}
搭建自己的shadowsocks代理服务器实现科学上网。内容包括VPS购买，连接VPS，一键搭建shadowsocks，开启bbr加速，客户端配置shaodowsocks。
{% endblockquote %}

#### 1. 购买VPS
VPS（Virtual private server，虚拟专用服务器），个人用来搭建一些博客，跑跑脚本足够了。今天的教程就用VPS来搭建属于自己的shaodowsocks，一个人独占一条线路。

Vultr是美国的一个VPS服务商。
##### 1.1 新用户注册
优惠注册链接：www.vultr.com
{% asset_img 001.png 注册 %}

##### 1.2 充值。
因为Vultr是按小时收费的。先充值才可以用。
验证并登录后我们会跳转到充值界面，或者从Billing->Make Patment进入：
{% asset_img 002.png 充值 %}

支持支付宝，微信等，很方便，充值10刀，按小时扣费，只要保证账户有余额，你的服务器就会一直运行。

##### 1.3 购买服务器
{% asset_img 003.png 购买服务器 %}
由于2.5刀的没有ipv4。所以我直接购买的3.5刀每月的。
{% asset_img 004.png 购买服务器选择 %}
点击开启ipv6。然后Deploy Now 就可以了。
{% asset_img 005.png 购买服务器 %}

##### 1.4 获取VPS登录信息
选择Deploy后，过个几分钟，就可以看到自己的服务器信息了，具体位置在Servers->Instances，点击选择你新建的实例：

{% asset_img 006.png 获取登录信息 %}

#### 2. 连接VPS
这时候开始连接我们的VPS。
##### 2.1 开启防火墙端口
这个和咱们国内的云服务器差不多。
点击菜单  Servers >> Firewall >> Add Firewall Group >> 输入描述 >> 编辑ipv4 规则。 >> Linked Instances >> 选择我们新建的vps。

{% asset_img 007.png 防火墙信息 %}

##### 2.2 测试是否被墙。
站内地址：{% post_link Test-外网是否被墙 %}


##### 2.3 Windows Xshell 连接。
在Servers Information 中。有ip 和 账号信息。直接连接就可以。


#### 3. 一键搭建shaodowsocks

##### 3.1 下载一键搭建ss脚本文件（直接复制这段代码运行即可）
    git clone https://github.com/YangXHero/flyzy2005-ss-fly.git
{% asset_img 009.png 下载git代码 %}
如果提示 `bash: git: command not found`，则先安装git：`yum install git -y`
##### 3.2 运行搭建ssr脚本代码
听说ssr是改进版本，可以伪装成自定义的http开头，让监控着认为你是在访问百度，将实际的内容进行加密。SS是纯加密流量。所以直接搭建ssr。

    ss-fly/ss-fly.sh -i flyzy2005.com 1024

##### 3.3 输入对应的参数
    
    密码：flyzy2005.com
    端口：1024

全部选择结束后，会看到如下界面，就说明搭建ssr成功了：

    Congratulations, ShadowsocksR server install completed!
    Your Server IP        :你的服务器ip
    Your Server Port      :你的端口
    Your Password         :你的密码
    Your Protocol         :你的协议
   

##### 3.4 相关操作命令
    修改配置文件：vim /etc/shadowsocks.json
    停止ss服务：ssserver -c /etc/shadowsocks.json -d stop
    启动ss服务：ssserver -c /etc/shadowsocks.json -d start
    重启ss服务：ssserver -c /etc/shadowsocks.json -d restart

##### 3.5 卸载
    ss-fly/ss-fly.sh -uninstall


#### 4. 一键开启BBR加速

BBR是Google开源的一套内核加速算法，可以让你搭建的shadowsocks/shadowsocksR速度上一个台阶，本一键搭建ss/ssr脚本支持一键升级最新版本的内核并开启BBR加速。

BBR支持4.9以上的，如果低于这个版本则会自动下载最新内容版本的内核后开启BBR加速并重启，如果高于4.9以上则自动开启BBR加速，执行如下脚本命令即可自动开启BBR加速：

    ss-fly/ss-fly.sh -bbr
{% asset_img 010.png bbr安装成功 %}
##### 4.1 判断bbr加速是否成功
判断BBR加速有没有开启成功。输入以下命令：
    
    sysctl net.ipv4.tcp_available_congestion_control
如果返回值为：
    
    net.ipv4.tcp_available_congestion_control = bbr cubic reno
后面有bbr，则说明已经开启成功了。

#### 5. 启动

* 有两种启动方式，建议使用配置文件的方式启动

执行`vim /etc/shadowsocks.json` 添加如下内容：

多用户配置如下：
```shell
{  
 "server":"0.0.0.0"，  
 "local_address": "127.0.0.1",  
 "local_port":1080,  
  "port_password": {  
     "8388": "password",  
     "8387": "password",  
     "8386": "password",  
     "8385": "password"  
 },  
 "timeout":300,  
 "method":"rc4-md5",  
 "fast_open": false  
}  
```
然后通过执行一下命令启动：

    ssserver -c /etc/shadowsocks.json -d start
    如果要停止运行，将命令中的start改成stop。

TIPS: 加密方式推荐使用rc4-md5，因为 RC4 比 AES 速度快好几倍，如果用在路由器上会带来显著性能提升。旧的 RC4 加密之所以不安全是因为 Shadowsocks 在每个连接上重复使用 key，没有使用 IV。现在已经重新正确实现，可以放心使用。更多可以看 issue。

#### 6.开机自启
编辑一下/etc/supervisord.conf文件，命令如下：
`vim /etc/supervisord.conf`

把下面的内容粘贴到文件尾部的空行处，然后保存：

    [program:shadowsocks]
    command=ssserver -c /etc/shadowsocks.json
    autostart=true
    autorestart=true
    user=root
    log_stderr=true
    logfile=/var/log/shadowsocks.log

接下来需要编辑一下/etc/rc.local文件，请执行以下命令：

    vi /etc/rc.local

请把以下内容粘贴到文件中部的空白处，然后保存

    service supervisord start

完成以上步骤后，重启之后，shadowsock会自动运行。







