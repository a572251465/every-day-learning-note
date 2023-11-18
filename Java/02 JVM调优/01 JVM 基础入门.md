# 一，基础入门

## 1，JVM 概述

```
一旦我们提到JVM就会忍不住跟 Java 联系到一起，总感觉他们相互依赖的，就好比JVM 天生就是为 Java服务的（其实刚开始确实是这样的，但是后来就不是了）。
```

（Java/ 的跨平台）

在真正的了解jvm之前，我们先来回顾下Java的跨平台特性（俗话说：“一处编译，到处运行”）。虽然我们编写的Java文件最终被编译为class文件，但是class文件本身是不能运行的。想要运行必须依赖 JVM这个平台（如下图）。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/0284942750bf4773a5c3b2d11255ee04.png)

> 通过上述图可以得知，class能运行在不同的平台上，取决于不同的平台都安装了JVM，所以说JVM本身就是跨平台的，所以运行在JVM上的语言也具有跨平台的特性。

（JVM 运行多种语言）

JVM 最初确实是为了JAVA 而服务的，后来越来越的语言实现了JVM的规范，所以他们都可以运行在JVM上，已经不仅限于Java了

可以理解为JVM 本身就是一个系统/ 平台。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/04c1a64f266b48bead725f6f3b04dcb6.png)

## 2，什么是JVM

> 其实很多人将JVM 以及JDK 傻傻分不清楚，那么我们通过这个小章节来讲述下JVM 和JDK之前的区别。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/8a6630a0f8c04ee585f8d76df337aede.png)

其实我们可以理解为最肤浅的包含关系。

如果您安装了JDK.  JDK / JRE / JVM

如果您安装了JRE. JRE/ JVM

- JDK 是开发所需要
- JRE 是运行所需要
- JVM 是运行的平台

（类似 如下图的关系）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/2c6af726257e4270a7b0b2b3a204b0aa.png)

## 3，Java 从编码到执行

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/4efc2ee767a04ea493f55fec3331d3f0.png)

上述的截图中解释了 Java 从编码到执行的过程

而我们今天的主角 `JVM` 就属于后半部分内容。其主要负责从 【class被加载】~【解释执行】等。

其实通过上述的截图也可以引出一些问题。

## 4，Java是编译执行/ 解释执行

其实无论我们说编译执行，还是解释执行，其实都不太对。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/5c09aa446c74481da39f1b55d58f9963.png)

（常用的部分是编译执行，动态的部分是解释执行）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/02608e560de740d1802df379a3f3b6b5.png)

## 5，常见的 JVM实现

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/30276/1699679986096/8122e300b03f4322957fd95582345e81.png)
