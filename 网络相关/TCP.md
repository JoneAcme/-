# TCP



# TCP基本知识

[TOC]

## 1. 定义

`Transmission Control Protocol`，即 传输控制协议。

属于 传输层通信协议，基于`TCP`的应用层协议有`HTTP`、`SMTP`、`FTP`、`Telnet` 和 `POP3`





## 2. 特点

- 面向连接、面向字节流、全双工通信、可靠
- 具体介绍如下：



![img](https://upload-images.jianshu.io/upload_images/944365-c77053c9881592ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/520)



## 3. 优缺点

- 优点：数据传输可靠
- 缺点：效率慢（因需建立连接、发送确认包等）

------

## 4. 应用场景（对应的应用层协议）

要求通信数据可靠时，即 数据要准确无误地传递给对方

> 如：传输文件：HTTP、HTTPS、FTP等协议；传输邮件：POP、SMTP等协议

- 万维网：`HTTP`协议
- 文件传输：`FTP`协议
- 电子邮件：`SMTP`协议
- 远程终端接入：`TELNET`协议

## 5. 报文段格式

- TCP虽面向字节流，但传送的数据单元 = 报文段
- 报文段 = 首部 + 数据 2部分
- TCP的全部功能体现在它首部中各字段的作用，故下面主要讲解TCP报文段的首部

> 1. 首部前20个字符固定、后面有4n个字节是根据需而增加的选项
> 2. 故 TCP首部最小长度 = 20字节



![img](https:////upload-images.jianshu.io/upload_images/944365-123333642e8eb31a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-4740db911582939f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/808)

示意图

------

## 6. 建立连接过程

- TCP建立连接需 **三次握手**
- 具体介绍如下



![img](https:////upload-images.jianshu.io/upload_images/944365-895493e20637d2b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/880)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-d148731fa16316be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-5527d827865f8d30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

**成功进行TCP的三次握手后，就建立起一条TCP连接，即可传送应用层数据**

> 注
>
> 1. 因 `TCP`提供的是全双工通信，故通信双方的应用进程在任何时候都能发送数据
> 2. 三次握手期间，任何1次未收到对面的回复，则都会重发

### <font color="#ff0000">特别说明：为什么TCP建立连接需三次握手？</font>

- 结论
  防止服务器端因接收了**早已失效的连接请求报文**，从而一直等待客户端请求，最终导致**形成死锁、浪费资源**
- 具体描述



![img](https:////upload-images.jianshu.io/upload_images/944365-1551b53e24060636.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/857)

## 7. 释放连接过程

- 在通信结束后，双方都可以释放连接，共需 **四次挥手**
- 具体如下



![img](https:////upload-images.jianshu.io/upload_images/944365-6162a7db50ebb9d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/950)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-91b079843a9e8235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图



![img](https:////upload-images.jianshu.io/upload_images/944365-82c3290a6135a610.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

### <font color="#ff0000">特别说明：为什么TCP释放连接需四次挥手？</font>

- 结论
  为了**保证通信双方都能通知对方 需释放 & 断开连接**

> 即释放连接后，都无法接收 / 发送消息给对方

- 具体描述



![img](https:////upload-images.jianshu.io/upload_images/944365-345dbc590bbcb19d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/816)



无差错传递等其他知识 参考链接：[TCP协议攻略](https://www.jianshu.com/p/65605622234b)





## 附：网络结构图



![](https://upload-images.jianshu.io/upload_images/944365-8f04f1321143fd6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

![](https://upload-images.jianshu.io/upload_images/944365-73d7d56b1f54d945.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/780)