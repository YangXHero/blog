---
title: Linux LVM 管理同一目录挂在多块磁盘
date: 2019-06-06 10:31:17
tags:
    - CentOS 管理
categories:
    - CentOS
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=470759757&auto=1&height=66"></iframe>
{% blockquote %}
遇到FastDFS文件服务器挂载盘会不够，就上网搜了一下挂载多块磁盘。
{% endblockquote %}
##### 如果想操作的挂载盘已挂载，则先卸载。
    umount -v /dev/data2
    umount -v /dev/data1
    
##### 创建PV - 物理卷
    pvcreate /dev/sdb1
    pvcreate /dev/sdc1
    pvscan
    
##### 创建VG - 卷组
    创建卷组并命名  vgfast 可自行修改
    vgcreate vgfast /dev/sdb1 
    添加盘 给卷组添加盘
    vgextend vgfast /dev/sdc1
##### 查看卷组
    vgs
##### 创建逻辑卷 - LV
    lvcreate -L 2.1T -n lvfast vgfast 
    
##### 格式化逻辑卷

    mkfs.ext4 /dev/vgdata/lvData  //格式化lvData为ext4格式。
##### 挂载目录
    mount /dev/vgfast/lvfast /fastdfs/
    