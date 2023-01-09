---
title: wireshark学习笔记----抓包网络原理
tags: wireshark
categories: wireshark
abbrlink: 685b2be6
date: 2019-07-28 02:00:47
---
抓包是我们常见的分析网络的一种方式，根据不同的抓包方式，网络原理也有所不同。
<!-- more -->
## 本机环境
---
   直接抓包本机网卡进出流量，这种抓包是直接抓取本地与互联网上交互的数据包，直接在本机上抓取。
## 集线器环境
---
   集线器环境下，因为都是以广播域的形式来传输数据包的，所以我们在一个集线器环境下，就能收到当前局域网下的所有数据包，就能直接抓取这些数据包来进行分析。
## 交换机环境
---
在交换机环境中，因为数据包不再以广播的形式发送，如果我们要抓取同一交换机环境下的其他主机的数据包，必须采取一些其他的策略。
### 1. 端口镜像
在交换机做端口镜像（SPAN技术）处理，将其他进入交换机的流量copy一份到本机
端口镜像实际上就是将交换机或者路由器与其他发给其他端口的数据转发一份到本端口，这样就可以获取交换机/路由器与其他端口交互的信息。
如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190728013434447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
在交换机与PC2和PC3的连接的端口上，可以看作图中的GE0/0/1和GE0/0/2上使用端口镜像，将PC2和PC3与交换机交互的数据包copy一份到交换机与PC1交互的端口上GE0/0/0，这样我们在PC1上就可以抓取交换机与PC2和PC3交互的数据包了。
### 2. ARP欺骗
 在交换机上，我们通过ARP协议来将ipv4地址转为MAC地址以发送数据到指定的设备上，而ARP欺骗，就是针对我们想要获取的信息，对源地址发出的ARP请求做出假的回复，使其发送到本机对应的MAC地址上。
 如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190728014040877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
这里PC2为了向PC3发送信息，此时又只知道IP3的IP地址，不知道其MAC地址，所以要通过ARP协议，发送一个广播帧，若IP地址为IP3，则返回ARP响应告知PC2 PC3的MAC地址，然而，PC1欺骗了PC2，告诉它自己的IP地址也是IP3，将自己的MAC地址发送给了PC2，这样一来，我们就能在PC1抓取PC2发送给PC3的数据包了。
### 3. MAC泛洪
MAC泛洪也是一种攻击方式，当我们在一台主机上，以足够快的速度发送去往位置目的的数据包，且数据包对应的MAC地址都不相同，那么就会将交换机/路由器上的路由表“爆掉”，使其他端口发送的数据包对应的MAC地址无法进入到交换机/路由器的路由表中，那么这些端口发送的信息都会变成以广播的形式发出，而我们就可以在本机上接收到这些广播信息了。
如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190728015527338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
首先PC1发送大量泛洪垃圾包，将交换机上MAC表中PC2和PC3的MAC地址挤出MAC表，MAC表更换后，PC2和PC3的数据在发送到交换机上后，发现没有对应的MAC地址可以发送，而且也无法加入新的MAC地址，只能将数据以广播的形式发送，而PC1就理所当然地得到了这些数据包，成功抓包。

文章参考[教程](https://www.bilibili.com/video/av46257479/?p=3)
上面地图片是使用**ensp**绘制的