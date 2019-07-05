---
title: 微信支付签名jar包问题
date: 2019-04-19 14:04:00
tags:
    - 微信支付
categories:
    - Java
---

#####针对jdk1.8.44以上版本
    
    将jre/lib/security/java.security文件中的
    将 #crypto.policy=unlimited
    改为 crypto.policy=unlimited
    其他不变，也不需要其他权限jar
    
######针对jdk1.8.44以下版本，
    将jre/lib/security/ 下 的 local_policy.jar和US_export_policy.jar替换为官方网站提供了JCE无限制权限策略文件的下载：
    JDK6的下载地址：
    http://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html
    JDK7的下载地址：
    http://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html
    JDK8的下载地址：
    http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html