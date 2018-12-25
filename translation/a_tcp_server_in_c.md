

# 用C实现一个TCP 服务器

> 用C写TCP服务器可以教会很多。理解如何实现TCP服务器以及涉及到的系统调用

[原文链接](https://ops.tips/blog/a-tcp-server-in-c/)

作者本人想要掌握Linux网络基本概念，觉得通过实现一个TCP服务器能更好的融会贯通。将整个流程写下来会强迫作者去掌握通常略过的细节。



如果读者你也好奇或者说只是想要个教程来实现一个TCP服务器，这篇文章适合你。

[TOC]

## 概括

简单说，有两个角色，客户端和服务端，服务端有代表机器本身的众所周知的iIP等待某一时刻被连接（这个地址可以通过DNS来获取）和一个众所周知的端口（代表这个机器上的某个服务）



在此时，啥也没发生，TCP的沟通机制在于建立连接，如果没有握手连接，数据不会发送

这就是在一开始发生的事情![](https://ops.tips/blog/-/images/tcp-connection-overview.svg)

一旦连接建立，服务端和客户端就开始去真正的执行他们的工作，比如说反复的同步数据

像HTTP/1.1这样的应用协议，已经定义好的服务端客户端模型，客户端会通过特定的方法来发起请求，服务端会处理这些请求来给返回结果

![](https://ops.tips/blog/-/images/tcp-interaction-http.svg)

在应用层，HTTP假定服务端能发送数据到客户端，反之亦然，如果有个管道以便于消息的发送，那就再好不过啦

> 假如你不熟悉这些概念，或者不了解网络层的划分，建议读者看计算机网络-自顶向下方法
>
> 这本书层层引导读者去从顶层向下理解网络概念

## socket(套接字)

在应用层与传输层之间的引导交互的接口就是socket

我在大学学到描述socket的比方：

> 假设房子就是服务器上的应用程序，socket就是他们的门，想要获取信息，总是要通过房门

![](https://ops.tips/blog/-/images/tcp-sockets.svg)

在C中，我们可以通过文件描述符(file descriptor, fd)来访问socket，通过socket(2)系统调用

这个借口可以用来做



## 创建一个socket

## (*)想一个CLOSED状态的socket写入会发生什么

## 绑定socket

## 让socket监听连接

## 接受链接

## 结论

## 资料

涉及到的书籍

- 计算机网络：自顶向下方法

- Unix网络编程 第一卷，套接字编程（UNP v1）

- [Linux/UNIX系统编程手册](Linux/UNIX系统编程手册)


值得推荐的文章

- [How Linux creates sockets and counts them](https://ops.tips/blog/how-linux-creates-sockets/)         
- [A UDP server and client in Go](https://ops.tips/blog/udp-client-and-server-in-go/)         
-  [How Linux allows TCP introspection](https://ops.tips/blog/how-linux-tcp-introspection/)         
-  [Using network namespaces and a virtual switch to isolate servers](https://ops.tips/blog/using-network-namespaces-and-bridge-to-isolate-servers/)         
- ​ [What is that /proc thing?](https://ops.tips/blog/what-is-that-proc-thing/)         

