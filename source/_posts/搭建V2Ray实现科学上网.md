---
title: 搭建V2Ray实现科学上网
date: 2020-06-05 10:32:07
tags:
    - CentOS 服务搭建
    - 科学上网
categories:
    - CentOS
---
{% blockquote %}
购买VPS 服务器就不阐述了，之前写过。
{% endblockquote %}
#### 安装V2ray
##### 1、安装curl
```text
yum update -y && yum install curl -y
```
##### 2、v2ray一键安装配置脚本
```text
bash <(curl -s -L https://git.io/v2ray.sh)
```
##### 3、安装步骤

1. 选择1进行安装
2. 传输协议默认TCP即可
3. 端口[xxx]，默认或者自行设置
4. 拦截广告，默认[N]
5. 是否配置 Shadowsocks ，默认[N]
    
{% asset_img 01.png 安装效果 %}

配置信息路径：/etc/v2ray/config.json

查看是否开机启动（默认v2ray开机启动）
```text
systemctl list-unit-files|grep v2ray
```
如果没有开机启动，输入下面指令
```text
systemctl enable v2ray
systemctl start v2ray
```
##### 4.打开防火墙
v2ray的端口默认是会被防火墙拦截的，一定要添加打开端口，这个很重要
     
```text
firewall-cmd --zone=public --add-port=端口号/tcp --permanent
firewall-cmd --reload
```
##### 5.开启BBR加速脚本
需要内核为4.1以上，如果服务器类型选择cent os 8那么应该已经自带了，我选用的centos 7，这里需要安装下，输入以下命令：
```text
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启

lsmod|grep bbr 查看BBR是否启用，如果输出tcp_bbr模块说明BBR已启动

开启bbr后速度提升非常明显， https://github.com/sivel/speedtest-cli.git

####V2Ray客户端配置
##### 1.Windows 客户端
|V2RayW	|V2RayW-v1.0.0-beta2|	[V2RayW Git](https://github.com/Cenmrev/V2RayW/releases)|
|---|---|---|
|V2RayN	|V2RayN-v2.53	|[V2RayN Git](https://github.com/2dust/v2rayN/releases)|
|V2RayS	|V2RayS_v1.0.0.3|	[V2RayS Git](https://github.com/Shinlor/V2RayS/releases)|

下面以Windows平台的V2RayW/V2rayN为例说明客户端的配置和使用方法：
###### V2RayW
   
1. 打开应用后右键托盘内的V2ray图标，点击“配置”
2. 在VMess服务器点击“增加”，在服务器信息分别输入 服务器ip、端口号、用户ID、额外ID，如下图：
    
{% asset_img 02.png 安装效果 %}

右击V2ray图标，选择自动模式(PAC)，然后就可以正确的上网了。
###### V2RayN

1. VPS控制台输入v2ray url 生成vemss URL，copy到内存
{% asset_img 03.png 安装效果 %}
2. 打开V2RayN客户端，点击左上角服务器——>从剪贴板导入URL
{% asset_img 04.png 安装效果 %}
3. 最后右键V2rayN客户端，选择http代理，推荐PAC，如果正常的话，那么已经可以科学上网了


#### V2Ray相关命令 
    v2ray info //查看 V2Ray 配置信息
    v2ray config //修改 V2Ray 配置
    v2ray link //生成 V2Ray 配置文件链接
    v2ray infolink //生成 V2Ray 配置信息链接
    v2ray qr //生成 V2Ray 配置二维码链接
    v2ray ss //修改 Shadowsocks 配置
    v2ray ssinfo //查看 Shadowsocks 配置信息
    v2ray ssqr //生成 Shadowsocks 配置二维码链接
    v2ray status //查看 V2Ray 运行状态
    v2ray start //启动 V2Ray
    v2ray stop //停止 V2Ray
    v2ray restart //重启 V2Ray
    v2ray log //查看 V2Ray 运行日志
    v2ray update //更新 V2Ray
    v2ray update.sh //更新 V2Ray 管理脚本
    v2ray uninstall //卸载 V2Ray
