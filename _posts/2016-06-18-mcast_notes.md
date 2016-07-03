---
layout: post
title:  "linux常用命令使用集锦"
date:   2015-02-10 4:51:53
categories: linux
---
### 组播编程遇到的几个问题记录

1 为了接收组播组数据，本地监听socket必须绑定一个端口，而接口ip必须是INADDR_ANY(ipv4)或者 inaddr6_any(ipv6)，或者是绑定一个组播IP（注意组播ip区分ipv4和ipv6）。但是，在windows平台下面，不能绑定组播地址，必须绑定wildcard ip。

2 在windows系统下面显示组播组信息
    
    netsh interface ip show joins
3 unix系统下面显示组播组信息

    netstat -ng

4 ipv6组播编程在苹果系统下面，只能用IPV6_JOIN_GROUP/IPV6_LEAVE_GROUP。不能用
IPV6_ADD_GROUP/IPV6_DROP_GROUP

5 在windows下发现sin6_scope_id 并不是local interface index

6 在multi-home的机器下组播编程，必须指定发送的local interface。方法是设定mreq.imr_interface(ipv4) 或者mreq6.ipv6mr_interface(ipv6)。因为不指定，系统会默认选择一个出口，这样系统选择的窗口可能网络不通。比如windows安装了vmware，有很多虚拟interface，有的可能是不通的。

### 参考

[1] <http://stackoverflow.com/questions/10692956/what-does-it-mean-to-bind-a-multicast-udp-socket>

[2] <https://technet.microsoft.com/en-us/library/cc759719(v=ws.10).aspx>

[3} <http://blog.csdn.net/wangmj518/article/details/7782917>


