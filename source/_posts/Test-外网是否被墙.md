---
title: Test 外网是否被墙
date: 2018-11-06 11:44:00
tags:
    - CentOS 服务搭建
categories:
    - CentOS
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=27580343&auto=1&height=66"></iframe>

{% blockquote %}
本篇用于测试外网Ip是否被墙。
{% endblockquote %}

#### 1. 全国 Ping 测试网页(https://www.ipip.net/ping.php)
如下图，输入ip之后点击ping，如果丢包率 100%，那肯定是被 Q 了，这种情况只能删除机器重建了。反之，那也不一定说明没有被 Q，接下来用下面两步继续检测。
{% asset_img 001.png ping测试 %}

#### 2. 国内外端口扫描测试(http://tool.chinaz.com/port)
如果出现下面情况，说明在国内该 IP 已经被封掉了，试试下一步去国外检测端口是否可用。
{% asset_img 002.png ping测试 %}
如果出现下面情况，说明国内并没有封掉该 IP。
{% asset_img 003.png ping测试 %}

#### 3. 国外测试(https://www.yougetsignal.com/tools/open-ports/)
{% asset_img 004.png ping测试 %}

* 如果上一步 22 端口是关闭状态，在这边检测是 open 状态，说明 IP 肯定是被封掉了，只能删除机器重建。
* 如果上一步 22 店口是关闭状态，这边检测也是 close 状态，那就要查看是不是服务器的防火墙把端口限制了。