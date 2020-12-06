# 一、入门



## 1、前言

“分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统”——《分布式系统原理与范型》

布式服务架构以及流动计算架构势在必行，亟需**一个治理系统**确保架构有条不紊的演进。(Dubbo)



**发展演变：**ORM(数据访问框架)——MVC(垂直应用系统)——RPC(分布式服务框架)——SOA(**Service Oriented Architecture**)。





## 2、Dubbo

RPC【Remote Procedure Call】是指**远程过程调用**，是一种**进程间通信方式**，他是一种技术的**思想**，而不是规范。它允许程序调用**另一个地址空间（通常是共享网络的另一台机器上）的过程或函数**，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。



RPC两个核心模块：**通讯**和**序列化**。



Apache Dubbo (incubating) |ˈdʌbəʊ| 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：**面向接口的远程方法调用**，**智能容错和负载均衡**，以及**服务自动注册和发现**。[Dubbo官网](http://dubbo.apache.org/zh-cn/)



**框架及各个组件如下：**

<<<<<<< HEAD
![](F:\dubbo-demo\dubbo-helloworld\img\dubbo-structrue)
=======
![](https://github.com/Woodyiiiiiii/dubbo-learning/blob/master/img/dubbo-structrue.png)
>>>>>>> c586f811f4191350eb1d1ca8d33310f32a3a3d67

* **服务提供者（Provider）：**暴露服务的服务提供方，服务提供者在启动时，向注册中心**注册**自己提供的服务
* **服务消费者（Consumer）: **调用远程服务的服务消费方，服务消费者在启动时，向注册中心**订阅**自己所需的服务，服务消费者，从提供者地址列表中，基于**软负载均衡算法**，选一台**提供者**进行调用，如果调用失败，再选另一台调用
* **注册中心（Registry）：**注册中心**返回服务提供者地址列表**给消费者，如果有变更，注册中心将基于**长连接**推送变更数据给消费者
* **监控中心（Monitor）：**服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心



Ø 调用关系说明

* 服务容器负责启动，加载，运行服务提供者。
* 服务提供者在启动时，向注册中心注册自己提供的服务。
* 服务消费者在启动时，向注册中心订阅自己所需的服务。
* 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
* 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
* 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。





## 3、dubbo入门



### (1)、zookeeper注册中心(Registry)

---

* zookeeper版本：3.4.11
* Windows10



zookeeper是一个**树型的目录服务**，支持变更推送，适合作为Dubbo服务的**注册中心**。



下载zookeeper，在bin(可执行的二进制文件目录)的文件目录输入命令cmd，打开命令框，输入zkServer.cmd启动zookeeper，如果显示无法找到zk.config文件，复制conf文件夹下的zoo_sample.conf文件，更名为zoo.conf，打开后可以设置dataDir(数据目录)为总目录下的data文件夹(新建)，用来保存数据。

然后进入bin文件夹，打开cmd，输入zkCli.cmd打开zookeeper客户端(端口号为2181)。

命令有：

* ls / ：列出根节点下的所有节点
* get /节点名 ：得到节点数据
* create -e /节点名 数据 ：创建新节点



**zookeeper的详细使用可看官网。**[Dubbo官网下zookeeper文档](http://dubbo.apache.org/zh-cn/docs/user/references/registry/zookeeper.html)



### (2)、管理控制台Dubbo-admin搭建(Monitor)

---

dubbo本身并不是一个服务软件。它其实就是一个**jar包能够帮你的java程序连接到zookeeper**，并**利用zookeeper消费、提供服务**。所以你不用在Linux上启动什么dubbo服务。

但是为了让用户更好的管理监控众多的dubbo服务，官方提供了一个**可视化的监控程序**，不过这个监控即使不装也不影响使用。



[Dubbo-admin下载地址](https://github.com/apache/dubbo-admin)

这是一个SpringBoot配置的dubbo-admin的jar包，下载解压后，打开dubbo-admin下的/main/resources/application.properties文件，确认是默认地址是本机的**2181端口**，然后使用mvn clean package在target包下生成项目jar包，cmd命令下输入java -jar dubbo-...启动项目jar包，然后输入localhost:7001就能看见可视化的dubbo管理控制台(确定zookeeper的服务端打开)，账号和密码都是root。



### (3)、创建提供者消费者工程

---

某个电商系统，订单服务需要调用用户服务获取某个用户的所有地址；

我们现在需要创建两个服务模块进行测试：

| 模块                | 功能           |
| ------------------- | -------------- |
| 订单服务web模块     | 创建订单等     |
| 用户服务service模块 | 查询用户地址等 |

测试预期结果：

​     订单服务web模块在A服务器，用户服务模块在B服务器，A可以远程调用B的功能

现在**服务提供者**是用户服务service模块，提供用户地址列表。



user-service-provider项目定义UserAddress类，包含用户地址等信息，然后在service包下根据参数用户ID返回用户列表；order-service-consumer项目的service包下根据参数用户ID调用远程provider项目的UserService类查询用户并展示。

根据dubbo的远程接口调用特性，将上述的接口函数和实体类统一放在gmall-interface项目中，接口实现类依旧。

要使远程调用成功，需要在maven中导入gmall-interface的GAV，所以把provider和consumer项目定义为gmall-interface的**子模块**，只要在gmall中定义新的module就可以实现。



### (4)、服务提供者配置&测试

---

导入Jar包，在mvnrep分别用com.alibaba下的dubbo，如果是2.6版本以上用curator(zookeeper的客户端)作为注册中心。



配置xml文件：

1. 注册dubbo:application应用
2. 使用dubbo:registry广播注册中心地址
3. 用dubbo:protocol表明双方通信用什么协议
4. 用dubbo:service声明需要暴露的服务接口，需要结合<bean/>

然后创建Spring容器，并启动，就可以在7001端口打开zookeeper看到服务提供方。





### (5)、服务消费者配置&测试

---

同理导入Jar包。



配置xml文件：

1. 注册dubbo:application应用
2. 使用dubbo:registry广播注册中心地址
3. 用dubbo:referenec声明需要**调用**的服务接口，用interface表明提供者的接口

然后在方法中使用Spring注解方式，并在xml中配置<context>扫描。





### (6)、监控中心_Simple Monitor——安装使用

---

在consumer.xml和provider.xml中标注标签：

<dubbo:monitor protocol="registry"/> —— 在注册中心监测发现

或者：<dubbo:monitor address="registry"/> —— 从地址监测发现



这样就可以打开端口观看。





### (7)、与SpringBoot整合

---

步骤：

1. 在pom文件中引入dubbo.starter
2. 服务提供者在application.properties中配置dubbo:application，dubbo:registry，dubbo:monitor，dubbo:protocol，并且使用注解@Service(com.alibaba下)注解要暴露的服务(实现类)，最后加上@EnableDubbo开启dubbo注解
3. 服务消费者在application.properties中配置dubbo:application，dubbo:registry，dubbo:monitor，并且使用注解@Reference(com.alibaba下)注解要使用的服务(实现类)，最后加上@EnableDubbo开启dubbo注解





# 二、配置

**配置文件优先级：**

Java -D     >>     XML      >>      Properties



**重试次数：**

失败自动切换，当出现失败，重试其它服务器，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。

```
重试次数配置如下：
<dubbo:service retries="2" />
或
<dubbo:reference retries="2" />
或
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```





**超时时间：**
由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了避免超时导致客户端资源（线程）挂起耗尽，必须设置超时时间。

最好在服务提供者方配置。

覆盖规则：

1. 方法级配置别优于接口级别，即小Scope优先 
2. Consumer端配置 优于 Provider配置优于全局配置
3. 最后是Dubbo Hard Code的配置值（见配置文档）

reference method —— service method —— reference —— service —— consumer —— provider

```
Dubbo消费端：
全局超时配置
<dubbo:consumer timeout="5000" />

指定接口以及特定方法超时配置
<dubbo:reference interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:reference>



Dubbo服务端：
全局超时配置
<dubbo:provider timeout="5000" />

指定接口以及特定方法超时配置
<dubbo:provider interface="com.foo.BarService" timeout="2000">
    <dubbo:method name="sayHello" timeout="3000" />
</dubbo:provider>
```



