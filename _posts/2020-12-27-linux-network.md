---
title: linux-网络管理
date: 27-12-2020
categories: 
- linux
tags:
- note
---

## 一.查看网络配置

### 1.ifconfig
```
ifconfig  //root用户
/sbin/ifconfig  //普通用户
```

	显示网卡信息：ip地址/子网掩码/网卡mac地址/网卡发送接收包的数量

	lo：环回网卡：127.0.0.1

### 2.只查看某一个网卡的信息
```
ifconfig eth0
```
### 3.查看网卡的网线的连接状态
```
mii-tool eth0
```
### 4.查看网关
```
route -n //-n参数不解析主机名
```

	查看结果中包含默认网关和明细路由，

	明细路由指“访问固定ip时，使用另一个固定网关”，即静态路由
	
### 5.查看主机名
```
hostname
```

## 二.修改网络配置

### 1.修改网卡ip

	ifconfig <接口> <IP地址> \[netmask 子网掩码]
```
ifconfig eth0 192.168.1.31 netmask 255.255.255.0
```
### 2.up网卡

	ifup <接口>
```
ifup eth0
```

### 3.down网卡

	ifdown <接口>
```
ifdown eth0
```

### 4.修改默认网关

	route add default gw <网关ip>
```
route del default gw 192.168.0.2 //需要先删除默认网关
route add default gw 192.168.0.1
```

### 5.修改静态路由

	route add -host <指定ip> gw <网关ip>
```
route add -host 10.221.31.5 gw 193.168.0.2
```

### 6.修改指定网段的静态路由

	route add -net <指定网段> netmask <子网掩码> gw <网关ip>
```
route add -net 10.0.0.0 netmask 255.255.255.0 gw 192.168.0.1
```

## 三.网络故障排除命令

### 1.ping

	ping不通的可能原因：网络中断/对方有防火墙/。。。

	支持域名和ip

### 2.traceroute

	辅助ping命令，追踪路由的每一跳
```
traceroute -w 1 www.baidu.com //-w 1表示在某个节点如果长时间timeout，只等待1秒钟
```

### 3.mtr

	辅助ping命令，检查到目标主机间是否有数据丢包

### 4.nslookup

	域名对应的ip是什么，可通过该命令查看
```
nslookup www.baidu.com //可以得到该域名对应的ip
```

	查看dns
```
nslookup
server
exit
```

### 5.telnet

	主机检查通了，但是服务仍然访问不了，可以使用telnet检查端口的连接状态

	需要首先安装telnet：yum install telnet -y

	telnet www.baidu.com 80

### 6.tcpdump
```
tcpdump -i any -n port 80 //查看所有网卡的访问80端口的数据包，并且域名解析成ip
tcpdump -i any -n host 10.0.0.1
tcpdump -i any -n host 10.0.0.1 and port 80
tcpdump -i any -n host 10.0.0.1 and port 80 -w /tmp/afile //将抓包数据保存在指定文件中
```

### 7.netstat
```
netstat -ntpl //n表示域名解析成ip；t表示显示tcp包；p表示显示该端口对应的进程；l表示显示监听状态的进程
```

### 8.ss

	与netstat使用方法类似
