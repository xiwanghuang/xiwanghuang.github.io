---
title: NS2学习笔记（三）
date: 2019-06-03 11:52:37
tags:
---
# NS2节点
节点是网络拓扑的重要组成部分，是NS2符合网络组件的一个大类。这里对应NS手册的第五章。
## 节点基本元素
建立节点基本命令：
```tcl
set ns [new Simulator]
$ns node
```
一个（单播）节点的典型结构包含两个TclObject：一个地址分类器（classifier_）和一个端口分类器（dmux_）。这些分类器的功能是将传入的包分派至正确的代理或者链路出口去。  
每个节点至少包含以下部分：
* 一个地址或（id_），初始值为0，当节点建立时，模拟的名字控件将自动加1；
* 一个邻居链表（neighbor_）；
* 一个代理链表（agnet_）;
* 一个节点类型识别器（nodetype_）；
* 一个路由模块


建立多播节点时需要进行设置  
`set ns [new Simulator -multicast on]`
## 设置节点
### 控制函数
* $node entry：返回节点的入口指针
* $node reset：重新设置节点上的所有代理
### 地址和端口号管理
* $node id：返回节点的节点号，该号码是在类Simulator用方法$ns node创建每个节点自动生成和赋值的
* $node agent(port)：返回一个端口号为port的代理的句柄。若代理不存在则返回空字符串。
* add-route和add-toutes：单播的时候用来增加路由路径从而产生classifier_。使用语法为`$node add-route <destination id> <TclObject>`。TclObject是dmux_的入口，节点的多路复用端口。add-routes用于已添加多重路由路径到同一个目的地。
* delete-routes()
* init-routing()和rtObject()。前者设置实例变量mutiPath_，使之与同名的类变量相同。同时给该节点的路由控制器对象增加一个实例变量rtObject_；后者返回该节点的路由对象的句柄。
  
最后，当节点上的一个链路关联改变状态时，过程intf-changed()可以被网络动态代码激活。
### 代理管理
attach()：将给定的代理加入到一个agents_链表中，给该代理分配一个端口号并设置源地址。设置代理的指针为它的入口。  
detach()：移除代理。
### 添加邻居
add-neighbor()
neighbor()
## 节点设置接口
$ns node-config -addressType hier
## Classfier
Classfier是NS2基本网络组件的一个大类。派生类有地址分类器和多播分类器等。当接收到一个分组包是，节点的任务就是检查包的域，通常是它的目的地址，优势也可能是它的源地址。得到一个指针指向所要到的出口的接口对象，也就是包的下一步接收器