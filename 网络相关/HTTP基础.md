# HTTP基础

#  HTTP知识

[TOC]

## 1.计算机网络各层基础

计算机网络各层：

![](https://upload-images.jianshu.io/upload_images/944365-8f04f1321143fd6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)

计算机网络各层作用：

![](https://upload-images.jianshu.io/upload_images/944365-73d7d56b1f54d945.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/780)

## 2. HTTP简介

![](https://upload-images.jianshu.io/upload_images/944365-a6c6e24b7bb7bfc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)



**HTTP 和TCP:**

**<font color="#ff0000">HTTP协议属于应用层，是基于TCP传输层协议的</font>**



## 3. 工作方式

- `HTTP`协议采用 **请求 / 响应** 的工作方式
- 具体工作流程如下：



![img](https://upload-images.jianshu.io/upload_images/944365-28e4020dd64411b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/280)

## 4. HTTP报文详解

- `HTTP`在 应用层 交互数据的方式 = 报文
- `HTTP`的报文分为：请求报文 & 响应报文

> 分别用于 发送请求 & 响应请求时

### 4.1 请求报文

#### 4.1.1 报文结构

- `HTTP`的请求报文由 **请求行、请求头 & 请求体**组成，如下图



![img](https:////upload-images.jianshu.io/upload_images/944365-76f625b54c1039be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/580)

示意图

- 下面，将详细介绍每个组成部分

#### 4.1.2 结构详细介绍

##### 组成1：请求行

- 作用
  声明 请求方法 、主机域名、资源路径 & 协议版本
- 结构
  请求行的组成 = 请求方法 + 请求路径 + 协议版本

> 注：空格不能省



![img](https:////upload-images.jianshu.io/upload_images/944365-ee23b153f6d654c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

请求行的组成

- 组成介绍



![img](https:////upload-images.jianshu.io/upload_images/944365-332ccda4eb8625bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

> 此处特意说明GET、PSOT方法的区别：



![img](https:////upload-images.jianshu.io/upload_images/944365-acce2f2323fd28a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

示意图

- 示例
  设：请求报文采用`GET`方法、 `URL`地址 = [http://www.tsinghua.edu.cn/chn/yxsz/index.htm](https://link.jianshu.com/?t=http%3A%2F%2Fwww.tsinghua.edu.cn%2Fchn%2Fyxsz%2Findex.htm)；、`HTTP1.1`版本

则 请求行是：`GET /chn/yxsz/index.htm HTTP/1.1`

##### 组成2：请求头

- 作用：声明 客户端、服务器 / 报文的部分信息
- 使用方式：采用**”header（字段名）：value（值）“**的方式
- 常用请求头
  **1. 请求和响应报文的通用Header**



![img](https:////upload-images.jianshu.io/upload_images/944365-254eb5a7db3d3fe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

请求和响应报文的通用Header

**2. 常见请求Header**



![img](https:////upload-images.jianshu.io/upload_images/944365-22f107afd0839c1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

常见请求Header

- 举例：
  (URL地址：[http://www.tsinghua.edu.cn/chn/yxsz/index.htm](https://link.jianshu.com/?t=http%3A%2F%2Fwww.tsinghua.edu.cn%2Fchn%2Fyxsz%2Findex.htm)）
  Host：[www.tsinghua.edu.cn](https://link.jianshu.com/?t=http%3A%2F%2Fwww.tsinghua.edu.cn)(表示主机域名）
  User - Agent：Mozilla/5.0 (表示用户代理是使用Netscape浏览器）

##### 组成3：请求体

- 作用：存放 需发送给服务器的数据信息

> 可选部分，如 `GET请求`就无请求数据

- 使用方式：共3种



![img](https:////upload-images.jianshu.io/upload_images/944365-6a361cc6960eb113.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/860)

示意图

**至此，关于请求报文的请求行、请求头、请求体 均讲解完毕。**

#### 4.1.3 总结

- 关于 请求报文的总结如下



![img](https:////upload-images.jianshu.io/upload_images/944365-63e390481e92aebe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/873)

示意图

- 请求报文示例



![img](https:////upload-images.jianshu.io/upload_images/944365-806653dd8c7df0e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/480)





### 4.2 HTTP响应报文

#### 4.2.1 报文结构

- `HTTP`的响应报文包括：状态行、响应头 & 响应体



![img](https:////upload-images.jianshu.io/upload_images/944365-e74ef9116a1b5df8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/580)

示意图

- 其中，响应头、响应体 与请求报文的请求头、请求体类似
- 这2种报文最大的不同在于 状态行 & 请求行

下面，将详细介绍每个组成部分

#### 4.2.2 结构详细介绍

##### 组成1：状态行

- 作用
  声明 协议版本，状态码，状态码描述
- 组成
  状态行有协议版本、状态码 &状态信息组成

> 其中，空格不能省



![img](https:////upload-images.jianshu.io/upload_images/944365-834e3a1b316f265c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

状态行组成

- 具体介绍



  ![img](https:////upload-images.jianshu.io/upload_images/944365-bd27e7365b0b2855.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

  示意图

- 状态行 示例
  `HTTP/1.1 202 Accepted`(接受)、`HTTP/1.1 404 Not Found`(找不到)

##### 组成2：响应头

- 作用：声明客户端、服务器 / 报文的部分信息
- 使用方式：采用**”header（字段名）：value（值）“**的方式
- 常用请求头
  **1. 请求和响应报文的通用Header**



![img](https:////upload-images.jianshu.io/upload_images/944365-254eb5a7db3d3fe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

请求和响应报文的通用Header

**2. 常见响应Header**



![img](https:////upload-images.jianshu.io/upload_images/944365-a9f0a212c4ea72b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

常见响应Header

##### 组成3：响应体

- 作用：存放需返回给客户端的数据信息
- 使用方式：和请求体是一致的，同样分为：任意类型的数据交换格式、键值对形式和分部分形式



![img](https:////upload-images.jianshu.io/upload_images/944365-9e829a9edda9ceb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/860)

示意图

#### 4.2.3 响应报文 总结



![img](https:////upload-images.jianshu.io/upload_images/944365-cc24a9b8effcbd42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/832)

示意图

### 4.3 总结

下面，简单总结两种报文结构



![img](https:////upload-images.jianshu.io/upload_images/944365-57aec599cfef3f0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)



## 5.HTTPS

### 5.1 HTTP的缺点

1. 通信使用明文(不加密), 内容可能会被窃听
2. 不验证通信方的身份, 因此有可能遭遇伪装
3. 无法证明报文的完整性, 所有有可能已遭篡改



### 5.2  HTTPS

<font color="#ff0000">**HTTP+加密+认证+完整性保护 = HTTPS**</font>

HTTPS并非是应用层的一种新协议. 只是**HTTP通信接口部分用SSL**(Secure Socket Layer)和TLS (Transport Layer Security) **协议替代**而已.

通常, HTTP直接和TCP通信, 当使用SSL时, 演变成了先和SSL通信, 再由SSL和TCP通信了, 简而言之, 所谓HTTPS, 其实就是**身披SSL协议的这层外壳的HTTP**.

在采用SSL后, HTTP就拥有了HTTPS的加密, 证书和完整性的保护这些功能.

**SSL是独立于HTTP的协议**, 所有不光是HTTP协议, 其他运行在应用层的SMTP(邮件协议)和Telnet等协议均可配合SSL协议使用. 可以说SSL是当今世界上应用最广泛的网络安全技术.

### 5.3 SSL 非对称加密

**1、客户端发起HTTPS请求**

这个没什么好说的，就是用户在浏览器里输入一个https网址，然后连接到server的443端口。

**2、服务端的配置**

采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作，也可以向组织申请，区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面(startssl就是个不错的选择，有1年的免费服务)。

这套证书其实就是一对公钥和私钥，如果对公钥和私钥不太理解，可以想象成一把钥匙和一个锁头，只是全世界只有你一个人有这把钥匙，你可以把锁头给别人，别人可以用这个锁把重要的东西锁起来，然后发给你，因为只有你一个人有这把钥匙，所以只有你才能看到被这把锁锁起来的东西。

**3、传送证书**

这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等等。

**4、客户端解析证书**

这部分工作是有客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。

如果证书没有问题，那么就生成一个随机值，然后用证书对该随机值进行加密，就好像上面说的，把随机值用锁头锁起来，这样除非有钥匙，不然看不到被锁住的内容。

**5、传送加密信息**

这部分传送的是用**证书加密后的随机值**，目的就是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。

**6、服务段解密信息**

服务端用私钥解密后，得到了客户端传过来的随机值(私钥)，然后把内容通过该值进行对称加密，所谓对称加密就是，将信息和私钥通过某种算法混合在一起，这样除非知道私钥，不然无法获取内容，而正好客户端和服务端都知道这个私钥，所以只要加密算法够彪悍，私钥够复杂，数据就够安全。

**7、传输加密后的信息**

这部分信息是服务段用私钥加密后的信息，可以在客户端被还原。

**8、客户端解密信息**

客户端用之前生成的私钥解密服务段传过来的信息，于是获取了解密后的内容，整个过程第三方即使监听到了数据，也束手无策。



## 6.`HTTP1.1` 与 `HTTP1.0` 的区别



`Http1.1`比 `Http1.0`多了以下优点：

- 引入持久连接，即 在同一个`TCP`的连接中可传送多个`HTTP`请求 & 响应
- 多个请求 & 响应可同时进行、可重叠
- 引入更加多的请求头 & 响应头

> 如 与身份认证、状态管理 & `Cache`缓存等机制相关的、`HTTP1.0`无`host`字段



![](https://upload-images.jianshu.io/upload_images/944365-820f955afd57185f.png?imageMogr2/auto-orient/)

## 7.`HTTP` 处理长连接的方式

![](https://upload-images.jianshu.io/upload_images/944365-1ae8c95acbb1b364.jpg?imageMogr2/auto-orient/)