---
title: [docker swarm 遇到的一次坑] 
categories: docker
date: 2017-07-23 00:46:11
tags: [docker swarm]
---

# docker swarm network 故障

## 故障描述
DpInc 整个系统跑在docker swarm mode，在第30天时出现sourcemysql容器(容器所在机器名swarm02)不能访问schemaregistry容器(容器所在机器swarm00)的8081端口。

![docker](/img/docker/network-1.png)

```
$ swarm-01 可以访问在```swarm-00``` 上的```container-1``` 服务
$ swarm-02 可以访问在```swarm-00``` 上的```container-2``` 服务
$ swarm-02 访问在```swarm-00``` 上的```container-1``` 服务 
```

## 排错过程

1. 检查```swarm00```机器 iptables``` iptables -nvL | grep "8081" ``` 查看机器上8081端口是否被reject
```
 3538  250K ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8081
 2747  222K ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED tcp spt:8081
```
iptables规则正常

2. 检查 swarm00 机器 container-1 iptables 和 router 正常
```
# docker inspect 9a556e2998b5 | grep /var/run
"SandboxKey": "/var/run/docker/netns/7ca58b86b82c",

# ip netns exec 7ca58b86b82c ip route show
default via 172.19.0.1 dev eth1
10.0.0.0/24 dev eth0  proto kernel  scope link  src 10.0.0.31
10.255.0.0/16 dev eth2  proto kernel  scope link  src 10.255.0.18
172.19.0.0/16 dev eth1  proto kernel  scope link  src 172.19.0.8

# ip netns exec 7ca58b86b82c iptables -nvL
Chain INPUT (policy ACCEPT 36M packets, 2243M bytes)
 pkts bytes target     prot opt in     out     source               destination
11182  903K ACCEPT     tcp  --  *      *       0.0.0.0/0            10.255.0.18          tcp dpt:8081 ctstate NEW,ESTABLISHED

Chain OUTPUT (policy ACCEPT 19M packets, 1662M bytes)
 pkts bytes target     prot opt in     out     source               destination
 8758  695K ACCEPT     tcp  --  *      *       10.255.0.18          0.0.0.0/0            tcp spt:8081 ctstate ESTABLISHED
```

3. 抓取 swarm02 container访问 swam00 container 数据包
```
更换阿里源提速
# echo 'deb http://mirrors.aliyun.com/debian jessie main' > /etc/apt/sources.list
# apt-get update && apt-get install tcpdump
# tcpdump -i eth0 -w /tmp/tcpdump.pack
```
发现数据异常
![tcpdump](/img/docker/tcpdump.jpeg)

问出处在握手包上，每次 swarm02 主动握手，都会被 rst , rst 包通常情况出现在:
1. 端口未打开 
2. 请求超时
3. 提前关闭 
4. 其他原因 http://windtear.net/2009/10/iptables_drop_reset.html  
排除  1 2 ,因为 swarm-02 可以正常访问，可能出现在 3 , 4 上，提前关闭不太能，因为有机器能访问swarm00，使用 4 给出链接解决
```
简单解决是忽略这种类型的全部 RST 包
iptables -I INPUT -p tcp --tcp-flags SYN,FIN,RST,URG,PSH RST -j DROP
```

## 是否完全解决
否  
调研了下关于tcp rst 方面的知识，很少有出现在tcp 握手时刻被rst的，引发问题真正原因可能是docker swarm 的bug

## 最终处理方案
- 升级了docker，新版本对network有改动，有人提tcp rst相关的issue
- 持续观察，是否还会出现问题
