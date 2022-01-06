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
![20201123003749172](https://user-images.githubusercontent.com/65861540/148330042-d9602450-375b-48a8-ad9b-c713133520f6.png)

### 实作一 熟悉 Ethernet 帧结构

使用 Wireshark 任意进行抓包，熟悉 Ethernet 帧的结构，如：目的 MAC、源 MAC、类型、字段等。
![image-20211210163032176](https://user-images.githubusercontent.com/65861540/148330166-93ecfc0e-47ac-41b2-a550-e9e7c961f74a.png)

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

![image-20220106100252640](https://user-images.githubusercontent.com/65861540/148330217-c434e0fc-ac0a-433c-ac1b-2f548a5fedfe.png)

> Destination MAC: `3c:91:80:45:8a:f7`(目的MAC)
>
> Source MAC:  `94:b8:6d:a7:38:9d`    （源MAC）
>
> Destination MAC是同一子网内另一台主机的物理地址
>
> Source MAC是本机的物理地址
>

2.   然后 `ping qige.io` （或者本子网外的主机都可以），同时用 Wireshark 抓这些包（可 icmp 过滤），记录一下发出帧的目的 MAC 地址以及返回帧的源 MAC 地址是多少？这个 MAC 地址是谁的？

![image-20211210171000823](https://user-images.githubusercontent.com/65861540/148330350-b2362862-e696-4114-8020-7c5786cf3701.png)

> Destination MAC: `9a:50:2a:e0:ef:4e`   
>
> Source MAC: `94:b8:6d:a7:38:9d`    
>
> Destination MAC是本机所在子网的网关的物理地址
>
> Source MAC是本机的物理地址

3. 再次 `ping www.cqjtu.edu.cn` （或者本子网外的主机都可以），同时用 Wireshark 抓这些包（可 icmp 过滤），记录一下发出帧的目的 MAC 地址以及返回帧的源 MAC 地址又是多少？这个 MAC 地址又是谁的？

![image-20211210173151255](https://user-images.githubusercontent.com/65861540/148330393-8c36d038-7e7d-4115-9565-69378382aaaf.png)

>Destination MAC: `9a:50:2a:e0:ef:4e`   
>
>Source MAC: `94:b8:6d:a7:38:9d`   
>
>Destination MAC是本机所在子网的网关的物理地址
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

   ![image-20220106102459236](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106102459236.png)

   > Destination MAC`ff:ff:ff:ff:ff:ff`(Broadcast)

   ![image-20220106102916165](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106102916165.png)

   ![image-20220106103054189](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106103054189.png)

   > Destination MAC: `94:b8:6d:a7:38:9d`   
   >
   > Source MAC: `3c:91:80:45:8a:f7`    
   >
   > Destination MAC是本机的物理地址
   >
   > Source MAC是与本机在同一个子网的计算机的物理地址

3. 再次使用 `arp -d *` 命令清空 arp 缓存

4. 然后 `ping qige.io` （或者本子网外的主机都可以），同时用 Wireshark 抓这些包（可 arp 过滤）。查看这次 ARP 请求的是什么，注意观察该请求是谁在回应。

![image-20220106104126979](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106104126979.png)

> 此次ARP请求是在询问子网网关的MAC地址，该请求由网关回应

> ✎ **问题**
>
> 通过以上的实验，你应该会发现，
>
> 1. ARP 请求都是使用广播方式发送的
> 2. 如果访问的是本子网的 IP，那么 ARP 解析将直接得到该 IP 对应的 MAC；如果访问的非本子网的 IP， 那么 ARP 解析将得到网关的 MAC。
>
> 请问为什么？ 
>
> 如果访问的是所处的本网子网的ip，ARP 缓存中没有该 ip，那么就会发送一个广播，在子网中找寻这个ip，如果有 那么ARP解析协议将会直接得到该ip对应的Mac地址；如果访问的是非本子网的ip，那么ARP解析将直接得到网关的Mac地址。因为要想访问对方，在处于同一子网的条件下，应该知道对方的Mac地址，但是不处于同一子网，就需要对方所处子网网关的Mac地址。

## 网络层

### 实作一 熟悉 IP 包结构

使用 Wireshark 任意进行抓包（可用 ip 过滤），熟悉 IP 包的结构，如：版本、头部长度、总长度、TTL、协议类型等字段。

![image-20220106104650231](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106104650231.png)

✎ **问题**

为提高效率，我们应该让 IP 的头部尽可能的精简。但在如此珍贵的 IP 头部你会发现既有头部长度字段，也有总长度字段。请问为什么？

> ip头部长度字段和总长度字段是为了方便将上层将ip包里的数据取出来，从而可以让明白包的开始

### 实作二 IP 包的分段与重组

根据规定，一个 IP 包最大可以有 64K 字节。但由于 Ethernet 帧的限制，当 IP 包的数据超过 1500 字节时就会被发送方的数据链路层分段，然后在接收方的网络层重组。

缺省的，`ping` 命令只会向对方发送 32 个字节的数据。我们可以使用 `ping 192.168.83.19 -l 2000` 命令指定要发送的数据长度。此时使用 Wireshark 抓包（用 `ip.addr == 192.168.283.19` 进行过滤），了解 IP 包如何进行分段，如：分段标志、偏移量以及每个包的大小等

![image-20220106105551457](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106105551457.png)

![image-20220106105531895](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106105531895.png)

✎ **问题**

分段与重组是一个耗费资源的操作，特别是当分段由传送路径上的节点即路由器来完成的时候，所以 IPv6 已经不允许分段了。那么 IPv6 中，如果路由器遇到了一个大数据包该怎么办？

> 转发或者直接丢弃该数据包

### 实作三 考察 TTL 事件

在 IP 包头中有一个 TTL 字段用来限定该包可以在 Internet上传输多少跳（hops），一般该值设置为 64、128等。

在验证性实验部分我们使用了 `tracert` 命令进行路由追踪。其原理是主动设置 IP 包的 TTL 值，从 1 开始逐渐增加，直至到达最终目的主机。

请使用 `tracert (-4) qige.io` 命令进行追踪，此时使用 Wireshark 抓包（用 `icmp` 过滤），分析每个发送包的 TTL 是如何进行改变的，从而理解路由追踪原理。

![image-20220106110757534](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106110757534.png)

![image-20220106111006527](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106111006527.png)

✎ **问题**

在 IPv4 中，TTL 虽然定义为生命期即 Time To Live，但现实中我们都以跳数/节点数进行设置。如果你收到一个包，其 TTL 的值为 50，那么可以推断这个包从源点到你之间有多少跳？

> 计算跳数，用与该数相差最小的2的n次方（一定要大于该数）与该数相减，所得到的就是跳数
>
> 此处为64-50=14
>
> 所以这个包从原点到我之间有14条

## 传输层

### 实作一 熟悉 TCP 和 UDP 段结构

1. 用 Wireshark 任意抓包（可用 tcp 过滤），熟悉 TCP 段的结构，如：源端口、目的端口、序列号、确认号、各种标志位等字段。

   ![image-20220106121957193](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106121957193.png)

2. 用 Wireshark 任意抓包（可用 udp 过滤），熟悉 UDP 段的结构，如：源端口、目的端口、长度等。

![image-20220106122137313](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106122137313.png)

✎ **问题**

由上大家可以看到 UDP 的头部比 TCP 简单得多，但两者都有源和目的端口号。请问源和目的端口号用来干什么？

> 源端口和目的端口是用来确认某一个应用程序，IP 只能到达子网网关，MAC 地址到达子网下的指定主机，而端口号是达到主机上的某个应用程序。

### 实作二 分析 TCP 建立和释放连接

1. 打开浏览器访问 qige.io 网站，用 Wireshark 抓包（可用 tcp 过滤后再使用加上 `Follow TCP Stream`），不要立即停止 Wireshark 捕获，待页面显示完毕后再多等一段时间使得能够捕获释放连接的包。

2. 请在你捕获的包中找到三次握手建立连接的包，并说明为何它们是用于建立连接的，有什么特征。

   > 它们的长度都很短。通过发出 SYN 信号请求连接，然后服务器端回应 ACK 确认收到请求，然后主机再发出一个确认信号。第一次握手时除了 SYN = 1 外其余的标志都为 0 ，第二次握手时除了 SYN = 1 且 ACK = 1 外其余的标志都为 0 ，第三次握手时除了 ACK = 1 外其余的标志都为 0

3. 请在你捕获的包中找到四次挥手释放连接的包，并说明为何它们是用于释放连接的，有什么特征。

   > 它们的长度都很短。这里四次挥手为什么只抓到了三个包呢？原始是将第二次、第三次挥手合并成了一个包，所以只看到了三个包。首先发出 FIN 信号请求断开，然后服务器端回应一个 ACK 确认信号，然后又发出一个 FIN 信号（这里将 ACK 和 FIN 合并成立一个包），然后主机回应一个 ACK 确认信号，即可断开连接。

✎ **问题一**

去掉 `Follow TCP Stream`，即不跟踪一个 TCP 流，你可能会看到访问 `qige.io` 时我们建立的连接有多个。请思考为什么会有多个连接？作用是什么？

> 开辟了多个通道加快了传输的速度

✎ **问题二**

我们上面提到了释放连接需要四次挥手，有时你可能会抓到只有三次挥手。原因是什么？

> 第二次和第三次挥手时发出的包合并成了一个

## 应用层

应用层的协议非常的多，我们只对 DNS 和 HTTP 进行相关的分析。

### 实作一 了解 DNS 解析

1. 先使用 `ipconfig /flushdns` 命令清除缓存，再使用 `nslookup qige.io` 命令进行解析，同时用 Wireshark 任意抓包（可用 dns 过滤）。

   ![image-20220106124045550](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106124045550.png)

   ![image-20220106123858962](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106123858962.png)

2. 你应该可以看到当前计算机使用 UDP，向默认的 DNS 服务器的 53 号端口发出了查询请求，而 DNS 服务器的 53 号端口返回了结果。

   ![image-20220106123942283](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106123942283.png)

   ![image-20220106123958064](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220106123958064.png)

3. 可了解一下 DNS 查询和应答的相关字段的含义

   > 1.QR：查询/应答标志。0表示这是一个查询报文，1表示这是一个应答报文
   > 2.opcode，定义查询和应答的类型。0表示标准查询，1表示反向查询（由IP地址获得主机域名），2表示请求服务器状态
   > 3.AA，授权应答标志，仅由应答报文使用。1表示域名服务器是授权服务器
   > 4.TC，截断标志，仅当DNS报文使用UDP服务时使用。因为UDP数据报有长度限制，所以过长的DNS报文将被截断。1表示DNS报文超过512字节，并被截断
   > 5.RD，递归查询标志。1表示执行递归查询，即如果目标DNS服务器无法解析某个主机名，则它将向其他DNS服务器继续查询，如此递归，直到获得结果并把该结果返回给客户端。0表示执行迭代查询，即如果目标DNS服务器无法解析某个主机名，则它将自己知道的其他DNS服务器的IP地址返回给客户端，以供客户端参考
   > 6.RA，允许递归标志。仅由应答报文使用，1表示DNS服务器支持递归查询
   > 7.zero，这3位未用，必须设置为0
   > 8.rcode，4位返回码，表示应答的状态。常用值有0（无错误）和3（域名不存在）清除缓存

✎ **问题**

你可能会发现对同一个站点，我们发出的 DNS 解析请求不止一个，思考一下是什么原因？

> DNS不止一个的原因是DNS解析过程是先从浏览器的DNS缓存中检查是否有这个网址的映射关系，如果有，就返回IP，完成域名解析；如果没有，操作系统会先检查自己本地的hosts文件是否有这个网址的映射关系，如果有，就返回IP，完成域名解析；如果没有，电脑就要向本地DNS服务器发起请求查询域名；本地DNS服务器拿到请求后，先检查一下自己的缓存中有没有这个地址，有的话直接返回；没有的话本地DNS服务器会从配置文件中读取根DNS服务器的地址，然后向其中一台发起请求；直到获得对应的IP为止

### 实作二 了解 HTTP 的请求和应答

1. 打开浏览器访问 qige.io 网站，用 Wireshark 抓包（可用http 过滤再加上 `Follow TCP Stream`），不要立即停止 Wireshark 捕获，待页面显示完毕后再多等一段时间以将释放连接的包捕获。

2. 请在你捕获的包中找到 HTTP 请求包，查看请求使用的什么命令，如：`GET, POST`。并仔细了解请求的头部有哪些字段及其意义。

   ![1690963-20201226160249152-832212079](C:\Users\Administrator\Desktop\新建文件夹\1690963-20201226160249152-832212079.png)

3. 请在你捕获的包中找到 HTTP 应答包，查看应答的代码是什么，如：`200, 304, 404` 等。并仔细了解应答的头部有哪些字段及其意义。

   ![1690963-20201226160259971-510063767](C:\Users\Administrator\Desktop\新建文件夹\1690963-20201226160259971-510063767.png)

   > 1. 200：交易成功；
   >    304：客户端已经执行了GET，但文件未变化；
   >    404：没有发现文件、查询或URl；

✍ 建议：

HTTP 请求和应答的头部字段值得大家认真的学习，因为基于 Web 的编程中我们将会大量使用。如：将用户认证的令牌信息放到头部，或者把 cookie 放到头部等。

✎ **问题**

刷新一次 qige.io 网站的页面同时进行抓包，你会发现不少的 `304` 代码的应答，这是所请求的对象没有更改的意思，让浏览器使用本地缓存的内容即可。那么服务器为什么会回答 `304` 应答而不是常见的 `200` 应答？

> 因为第一次访问时成功收到响应，就会返回200，浏览器这时会下载资源文件，记录response header和返回返回时间。浏览器第二次发送请求的时候，告诉浏览器我上次请求的资源现在还在自己的缓存中，如果你那边这个资源还没有修改，就可以不用传送应答体给我了。服务器根据浏览器传来的时间发现和当前请求资源的修改时间一致，就应答304，表示不传应答体了，从缓存里取，不一致返回200。
