#  Wireshark 实验

本部分按照数据链路层、网络层、传输层以及应用层进行分类，共有 10 个实验。需要使用协议分析软件 `Wireshark` 进行，请根据简介部分自行下载安装。

## 准备

请自行查找或使用如下参考资料，了解 `Wireshark` 的基本使用：

- 选择对哪块网卡进行数据包捕获
- 开始/停止捕获
- 了解 `Wireshark` 主要窗口区域
- 设置数据包的过滤
- 跟踪数据流

🌏 参考

1. [Wireshark官方文档](https://www.wireshark.org/docs/wsug_html/)
2. [Wireshark抓包新手使用教程](https://www.cnblogs.com/linyfeng/p/9496126.html)
3. [Troubleshooting with Wireshark](http://file.allitebooks.com/20160907/Troubleshooting with Wireshark.pdf)
4. [The Official Wireshark Certified Network Analyst Study Guide](http://file.allitebooks.com/20150724/Wireshark Network Analysis- The Official Wireshark Certified Network Analyst Study Guide, 2nd Edition.pdf)
5. [Wireshark Network Security](http://file.allitebooks.com/20190315/Wireshark Network Security.pdf)

## 数据链路层

​		数据链路层是OSI参考模型中的第二层，介乎于物理层和网络层之间。数据链路层在物理层提供的服务的基础上向网络层提供服务，其最基本的服务是将源自网络层来的数据可靠地传输到相邻节点的目标机网络层。
![数据链路层图片](https://img-blog.csdnimg.cn/20201123003749172.png#pic_center)

### 实作一 熟悉 Ethernet 帧结构

使用 Wireshark 任意进行抓包，熟悉 Ethernet 帧的结构，如：目的 MAC、源 MAC、类型、字段等。

![image-20211210163032176](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211210163032176.png)

>Ethernet帧格式包含目的MAC，源MAC，类型，数据，校验字段。从图中可以看到该帧的
>
>目的MAC为`00:74:9c:9f:40:13`，
>
>源MAC为`98:28:a6:0a:c3:be`。
>
>数据的类型是IPv4，当数据不足43字节时，就会进行填充，使用Padding表示的填充数据。

**问题**

你会发现 Wireshark 展现给我们的帧中没有校验字段，请了解一下原因。

> 帧格式中时包含校验字段的，但是Wireshark在抓包的时候，自动将校验字段过滤掉了，所以Wireshark向我们展示的帧中没有校验字段

### 实作二 了解子网内/外通信时的 MAC 地址

1. `ping` 你旁边的计算机（同一子网），同时用 Wireshark 抓这些包（可使用 icmp 关键字进行过滤以利于分析），记录一下发出帧的目的 MAC 地址以及返回帧的源 MAC 地址是多少？这个 MAC 地址是谁的？

![在这里插入图片描述](https://www.pianshen.com/images/633/60a5ba0e1215a8a7a7d3c5b355585aa1.png)

> Destination MAC: `04：d3：b0：f6：5c：e8`(目的MAC)
>
> Source MAC:  `b8：27：eb：e3：2d：e7`    （源MAC）
>
> Destination MAC是同一子网内另一台主机的物理地址
>
> Source MAC是本机的物理地址
>
> PS:此处截图使用的是网上截图，因为本人技术有限，并未解决防火墙等问题，无法实现同子网两台主机间相互ping通，为更好体现实验结果，故使用网上截图，此截图来自文章：[Wireshark实验——了解PDU - 程序员大本营 (pianshen.com)](https://www.pianshen.com/article/85782112867/)

2.   然后 `ping qige.io` （或者本子网外的主机都可以），同时用 Wireshark 抓这些包（可 icmp 过滤），记录一下发出帧的目的 MAC 地址以及返回帧的源 MAC 地址是多少？这个 MAC 地址是谁的？

![image-20211210171000823](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211210171000823.png)

> Destination MAC: `00:74:9c:9f:40:13`   
>
> Source MAC: `98:28:a6:0a:c3:be`    
>
> Destination MAC是重庆交通大学网关的物理地址
>
> Source MAC是本机的物理地址

3. 再次 `ping www.cqjtu.edu.cn` （或者本子网外的主机都可以），同时用 Wireshark 抓这些包（可 icmp 过滤），记录一下发出帧的目的 MAC 地址以及返回帧的源 MAC 地址又是多少？这个 MAC 地址又是谁的？

![image-20211210173151255](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211210173151255.png)

>Destination MAC: `00:74:9c:9f:40:13`   
>
>Source MAC: `98:28:a6:0a:c3:be`   
>
>Destination MAC是重庆交通大学网关的物理地址
>
>Source MAC是本机的物理地址

> ✎ **问题**
>
> 通过以上的实验，你会发现：
>
> 1. 访问本子网的计算机时，目的 MAC 就是该主机的
> 2. 访问非本子网的计算机时，目的 MAC 是网关的
>
> 请问原因是什么？
>
> 原因是网关作为一个子网的出入口，当一个子网想要和另外一个子网进行通信，就必须经过网关，才能够实现通信，对于同一子网中的终端进行通信，它们只要在子网中找到对应的目的地址就可以实现通信，不需要经过网关。


### 实作三 掌握 ARP 解析过程

1. 为防止干扰，先使用 `arp -d *` 命令清空 arp 缓存

![image-20211210174851060](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20211210174851060.png)

2. `ping` 你旁边的计算机（同一子网），同时用 Wireshark 抓这些包（可 arp 过滤），查看 ARP 请求的格式以及请求的内容，注意观察该请求的目的 MAC 地址是什么。再查看一下该请求的回应，注意观察该回应的源 MAC 和目的 MAC 地址是什么。

3. 再次使用 `arp -d *` 命令清空 arp 缓存

4. 然后 `ping qige.io` （或者本子网外的主机都可以），同时用 Wireshark 抓这些包（可 arp 过滤）。查看这次 ARP 请求的是什么，注意观察该请求是谁在回应。

> ✎ **问题**
>
> 通过以上的实验，你应该会发现，
>
> 1. ARP 请求都是使用广播方式发送的
> 2. 如果访问的是本子网的 IP，那么 ARP 解析将直接得到该 IP 对应的 MAC；如果访问的非本子网的 IP， 那么 ARP 解析将得到网关的 MAC。
>
> 请问为什么？ 

