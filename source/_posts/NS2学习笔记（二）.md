---
title: NS2学习笔记（二）——分裂对象模型
date: 2019-05-31 15:40:09
tags:
---
# 解释器相关类的介绍和用途
## Tcl类
封装了OTcl解释器的实例，并提供了访问解释器的方法，提供访问Tcl库的接口。该类提供如下操作方法：
* **1、获取Tcl实例句柄**  
  Tcl类只有一个实例，该实例在NS2启动时初始化，获取Tcl类的实例方法是  
  `Tcl& tcl = Tcl::instance();`
* **2、用来调用OTcl命令函数**
  获得Tcl解释器引用之后可通过这个引用来调用OTcl中的控制台命令，现有4中方法通过Tcl实例来调用OTcl命令，主要区别是调用函数的不同。
  ```c++
  tcl.eval(s)               //执行已经存储进tcl返回区的命令，并返回给结果变量
  tcl.eval(char* s)         //执行字符串s，并在tcl的结果变量中保存执行结果
  tcl.evalc(const cahr* s)  //首先把s存储进tcl的命令缓冲区，然后在执行s命令，并在结果变量中返回结果
  tcl.evalf(const char* s,...)  //类似于c语言中的printf命令，可以进行字符串的过滤，执行同tcl.evalc。
  ```
* **3、传递/返回OTcl命令运行的结果**  
  当C++程序中调用一个OTcl命令是，解释器将执行结果保存在私有成员变量tcl_->results中，用户必须用tcl.result(void)来获取该执行结果，若结果是字符串，必须被转换为适当的类型。
  ```c++
  tcl.evalc("Simulator set NumberInterfaces_");
  cahr *ni = tcl.result();
  if( atoi(ni)!=1 ) //将字符串转换成整数
    tcl.evalc(Simulator set NumberInterfaces_1);
  ```
  Tcl类的结果返回，与脚本执行的退出码也有所不同，例如
  ```c++
  if( strcmp(argv[1],"now") == 0 ){
      tcl.resultf("%.17g",clock());
      return TCL_OK;
  }
  ```
* **4、存储并查询TclObject对象**  
  在NS2中的Tcl命令解释器中保存了一个存储对象地址的哈希表，NS2为每一个在模拟过程中生成的TclObject类以及派生类的对象在该哈希表中保存了一个指针，用于快速查找这些对象。
  ```c++
  Tcl.enter(TclObjuct* o)   //在哈希表中加入一个对象
  Tcl.lookup(char* s)       //在哈希表中查询名字为s的对象并返回
  Tcl.remove(TclObject* o)  //在哈希表中删除该对象的地址
  ```
* **5、获的Tcl解释器的句柄，来对解释器进行直接访问**
  函数`Tcl.interp(void)`是Tcl的解释器句柄，可以对其进行修改，加入自己的解释器。
## TclObject类
TclObject类是OTcl/C++两个面向对象语言的类的基库，封装了绑定、跟踪和对相关命令的调用机制。
* 1、创建/清除模拟器组件的对象
  ```c++
  set tcp1 [new Agent/TCP]
  delete $tcp1
  ```
* 2、实现从C++类成员变量到OTcl类成员变量的绑定
  ```c++
  bind("distance_",&distance)   //绑定实数变量
  bind_time("lastSent_",&lastSessSent)   //绑定时间变量
  bind_bw       //绑定带宽变量
  bind_boll     //绑定布尔型变量
  ```
  以上一个是OTcl变量名，一个是对应C++成员变量地址。
* 3、实现变量跟踪
* 4、实现从C++类的成员函数到OTcl类的成员函数之间的一一对应  
    通过command()函数事项。由于command()是TclObject的成员函数，所以它的派生类的每个对象都可以重写该函数。
    ```c++
    /*
    argc:命令行参数个数
    *argv:命令行参数向量：argv[0]:cmd;argv[1]指定想要的操作;argv[2...argc-1]用户还指定的参数
    参数以字符串形式给出，若操作匹配成功则返回操作结果；若操作不匹配则调用父类command()一直到匹配为止，若都不匹配则返回错误信号
    */
    Int MobileNode::command(int argc,const char*const* argv)
    {
      Tcl&tcl = Tcl::instance();
      if( argc == 5 ){
        if( strcmp(argv[1],"setdest")==0 )
        {
          if( set_destination( atof(argv[2]),atof(argv[3]),atof(argv[4]) )<0 )
          return TCL_ERROR;
          return TCL_OK;
        }
      }
      return( SRMAgent::command(argc,argv) );
    }
    ```
## TclClass类
TclClass类用于注册编译，保持了编译分级的层次结构，同时给OTcl对象提供了创建C++对象的方法。
`set o [new B/A]`
对象o的解释构造函数在ns第一次启动时执行：
* 次构造函数以解释器类的名字B/A调用ABClass的构造函数；
* 然后，ABClass构造函数调用其父类TclClass的构造函数；
* TclClass构造函数存储类的名字，并将对象插入TclClass对象链中；
* 在simulator的初始化过程中，Tcl_AppInit(void)调用TclClass::bind(void)。bind()调用以解释类的名字作为参数的register{},register{}建立类层次，生成需要却还没有的类；
* 最后，bind()为新类定义实例过程create-shadow和delete-shadow，生成并返回对象o。
## TclCommand类
TclCommand类用于定义简单的全局解释命令。TclCommand类也是纯虚函数。需要派生类实现两个成员函数：构造函数和command()。
## EmbeddedTcl类
用户对脚本~tclcl/tcl-object.tck进行修改，增加tcl/lib的文件来对ns进行扩展。对于新文件的装载是由该类完成的。
```c++
void EmbeddedTcl::load(){
  Tcl::instance().evalc(code_);
}
```
## InstVar类
包含了从OTcl访问C++类成员变量的方法。
# OTcl与C++之间的连接
主要实现以下三个功能：
* 动态创建一个新的C++对象；
* 访问该C++对象的属性；
* 调用该C++对象的方法。
## 调用C++对象的方法
* 注册顶级命令  
  使用command方法
* 暴露C++对象方法